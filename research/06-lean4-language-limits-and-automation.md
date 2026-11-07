# 06 — Lean 4 language limits and automation (Phase 1b, full-read upgrade)

**Header counts:** [FULL] entries: 34 · [CODE] entries: 8 · Unverified leads (excluded from evidence): 24
**Access date for every entry:** 2026-11-07 (latest Lean seen: v4.36.0-rc2 of 2026-10-27; latest stable v4.35.0 of 2026-10-12; lean4 `master` inspected at 2026-11-06).
**Status:** evidence only, no verdict. Hypothesis (verbatim, CONVENTIONS.md): "LEAN4 cannot verify protocols AND therefore unable to check code implementation of these protocols."

## How to read this file

- Tags: `[P]` primary (Lean docs, release notes, authors' own papers/slides/blog, source code), `[S]` secondary (third-party analysis, forum threads, repos by unrelated authors). Depth: `[FULL]` = the whole document was retrieved to disk and the sections relevant to this file's scope were read line by line; sections outside scope (for example the unrelated chapters of a 200 KB reference page) were not read, and each entry says what was read. `[CODE]` = a shallow clone was inspected (clone since deleted for disk reasons; SHA recorded).
- Sub-claim tags: H1 expressiveness, H2 provability at scale, H3 implementation link, H4 generality, H5 inference ("therefore"), D definitions of "verify"/"check".
- Search snippets were used only to locate sources. Any number that appears only in a snippet or in a truncated part of a fetched page has been moved to "Unverified leads".
- Counting method for repo tables: crude `grep` over `*.lean` files (excluding `.lake/` and `build/`). Counts of `sorry`, `axiom`, `native_decide` etc. include occurrences inside comments, docstrings and test examples, so they are upper bounds on real uses.

---

## 1. Repo inspection table (rule 3)

| Repo (origin) | SHA (10) | Last commit | lean-toolchain | Licence | Lean files / LOC | sorry | admit | axiom decl | native_decide | implemented_by | extern | opaque | partial def |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| leanprover/lean4 (all `*.lean`, incl. tests) | a2f1a642ab | 2026-11-06 | `build/release/stage1` (dev tree) | Apache-2.0 | 7,951 / 1,196,658 | 3,106 | 137 | 21,754 (tests/docstrings) | 269 | 214 | 1,094 | 1,017 | 2,032 |
| leanprover/lean4 `src/` only | a2f1a642ab | 2026-11-06 | – | Apache-2.0 | 2,634 / 763,589 | 221 | – | 10 | 12 | 192 | 1,056 | 517 | 1,702 |
| verse-lab/veil | 57447722e7 | 2026-11-07 | leanprover/lean4:v4.35.0 | Apache-2.0 | 495 / 21,760 | 37 | 10 | 0 | 22 | 16 | 0 | 1 | 67 |
| Verified-zkEVM/VCVio | 0b69833ce5 | 2026-11-06 | leanprover/lean4:v4.35.0 | Apache-2.0 | 1,613 / 437,887 | 13 | 10 | 7 | 14 | 16 | 37 | 35 | 19 |
| konnov/leanda | f7abaa1d04 | 2026-07-28 | none at root (per-project) | MIT | 25 / 19,196 | 1 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| akkeshavan/tla-plus-lean-workflow | 9910342183 | 2026-06-29 | none at root; `VerificationContinuum/` pins v4.31.0 (README) | no LICENSE file found at root | 20 / 3,377 | 8 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| scidonia/leanin | 3015043d80 | 2026-10-31 | leanprover/lean4:v4.35.0 | Apache-2.0 | 78 / 21,376 | 0 | 1 | 13 | 0 | 0 | 20 | 3 | 49 |
| ufmg-smite/lean-smt | 8e85c2e25b | 2026-11-05 | leanprover/lean4:v4.35.0 | Apache-2.0 | 280 / 25,861 | 38 | 4 | 3 | 0 | 2 | 0 | 11 | 36 |
| leanprover-community/duper | 0e593cc2cb | 2026-11-02 | leanprover/lean4:v4.35.0 | Apache-2.0 | 134 / 24,904 | 41 | 0 | 80 | 0 | 0 | 1 | 15 | 131 |

Reading notes: the four real axiom declarations in lean4 `src/` core are `Quot.sound` (Init/Prelude.lean:543), `sorryAx` (Prelude.lean:794), `Classical.choice` (Prelude.lean:865) and `propext` (Init/Core.lean:1605); the other six matches are docstring or test examples (`Rtrans`, `FooFam.bar`, `Int.add_comm` demo, etc.). The 80 `axiom` matches in duper and 13 in leanin were not individually classified. For veil the figures include `Examples/` and tests. **AI-generation flags:** leanin has an `AGENTS.md` at the repository root (agent working rules; there is no `CLAUDE.md`) and its docs carry internal tags such as `[R]`; I judge it "AI-agent-assisted" on that basis, not on an explicit statement. leanda's README says Lean proofs for Tendermint were "transferred from TLAPS by Codex GPT 5.6". The tla-plus-lean-workflow repo is a single independent author's STTT submission and is not peer-reviewed (README says "Submitted to: Software Tools for Technology Transfer").

---

## 2. Lean core: effects, concurrency, timing, nondeterminism (sub-claims H1, D)

**E01 — Lean Reference: "IO / Logical Model"** · https://lean-lang.org/doc/reference/latest/IO/Logical-Model/ · [P] · [FULL] · H1, D
IO actions are modelled as a state monad over "the entire world" carried by an abstract token, described as a "description of a process"; non-terminating code is handled with `partial`, "treated as arbitrary constants"; the page says there is "no way for values to 'escape'" IO without clearly-marked unsafe operators. What it demonstrates: the logical model of IO is a (world-passing) state transformer, so IO programs are Lean terms. What it does not cover: any model of what the world does between calls, nor of threads.

**E02 — Lean Reference: "Tasks and Threads"** · https://lean-lang.org/doc/reference/latest/IO/Tasks-and-Threads/ · [P] · [FULL] · H1
Pure `Task.spawn` tasks are cancelled when the last reference is dropped; `IO.asTask` tasks continue and must poll `IO.checkCanceled` (cancellation is cooperative). Pool size is `LEAN_NUM_THREADS` or the logical processor count and is "not a hard limit"; priorities 0–8 plus `dedicated`; `Task.get` temporarily grows the pool. `Promise`, `Std.Channel`, `CloseableChannel`, `Std.Mutex` (re-entrant `atomically` is undefined behaviour), `Std.Condvar` exist. `Std.Mutex` is "similar to IO.Ref α, except that concurrent accesses are guarded by a mutex instead of atomic pointer operations and busy-waiting". Scope: a runtime API description, no memory model or scheduling-fairness guarantee is stated on the page.

**E03 — Lean Reference: "Asynchronous Programming" chapter** · https://lean-lang.org/doc/reference/latest/ (async chapter; page text 97 KB retrieved) · [P] · [FULL] (read: intro, monad definitions, task semantics, concurrency combinators; not read: per-function API listings for TCP/UDP/DNS) · H1
Verbatim: "The Async monad provides tools and abstractions for constructing asynchronous programs that can safely multiplex different sources of data."; "Async is most appropriate when a program may spend a significant amount of time waiting on external events or I/O."; "The most important feature of Async is event selection."; "An asynchronous computation runs on the current thread until it must wait for a result that is not yet available"; "Crucially, calling await on a task never blocks an OS-level thread."; "While tasks from IO.asTask are synchronous, occupying their worker thread until completed, tasks from EAsync.asTask release their worker threads at suspension points". Three monads (`BaseAsync`, `EAsync ε`, `Async`) mirror `BaseIO`/`EIO`/`IO`; `Async.concurrently`/`concurrentlyAll` run computations together and "a failure in one subcomputation does not stop the others". The page describes an executable runtime; it states no program logic or specification language for it.

**E04 — Lean Reference: "Timing" and "Random Numbers"** · https://lean-lang.org/doc/reference/latest/ (sections 22.8, 22.10) · [P] · [FULL] · H1
Timing API is `opaque`: `IO.sleep (ms : UInt32) : BaseIO Unit`, `IO.monoNanosNow`, `IO.monoMsNow` ("Monotonically increasing time since an unspecified past point… There is no relation to wall clock time"). Random numbers: `IO.setRandSeed`, `IO.rand lo hi`, `randBool` over a `RandomGen` class. These are runtime primitives; the page defines no real-time or probabilistic semantics.

**E05 — Lean Reference: "Run-Time Code"** · https://lean-lang.org/doc/reference/latest/ (chapter 12) · [P] · [FULL] · H1, H3
Compiled code uses reference counting; "The Task API provides the ability to write parallel and concurrent code. The runtime is responsible for scheduling Lean tasks across operating-system threads." (Landing page only; subsections beyond that were not part of this read.)

**E06 — Lean Reference: "Recursive definitions" (partial / unsafe sections)** · https://lean-lang.org/doc/reference/latest/ (Recursive Definitions) · [P] · [FULL] (read: sections on partial, unsafe, implemented_by; not read: structural/WF recursion mechanics) · H3, D
Verbatim: "Allowing arbitrary recursive function definitions would make Lean's logic inconsistent."; "Functions marked partial are treated as opaque constants by the kernel and are neither unfolded nor reduced. All that is required for soundness is that their return type is inhabited."; unsafe definitions "may freely make use of general recursion, and they may use features of Lean that break assumptions about its equational theory, such as primitives for casting (unsafeCast), checking pointer equality (ptrAddrUnsafe), and observing reference counts"; "any declaration that refers to an unsafe definition must itself be marked unsafe"; implementation substitution: "logical soundness is not at risk, but the behavior of programs written in Lean may diverge from their verified logical models if the unsafe implementation is incorrect." Consequence stated by the manual: a `partial def` cannot be reasoned about in the logic.

**E07 — Lean Reference: "Axioms"** · https://lean-lang.org/doc/reference/latest/ (Axioms) · [P] · [FULL] · D
Three standard axioms (`propext`, `Classical.choice`, `Quot.sound`); `sorryAx` implements `sorry`; "proofs by native evaluation introduce a dedicated axiom for each computation that is asserted by the compiled code. These axioms track proofs that depend on the correctness of the entire compiler, and not just on the much smaller kernel." "Lean cannot check whether new axioms are consistent."

**E08 — Lean Reference: "Validating a Lean Proof"** · https://lean-lang.org/doc/reference/latest/ (Validating) · [P] · [FULL] · D
Defines the checking procedure: a green tick means the statement elaborated and "the Lean kernel has accepted a proof"; it "is meaningful if one believes the formal theorem statement corresponds to its intended informal meanings and trusts the authors of the imported libraries". `#print axioms` should show only `propext`, `Classical.choice`, `Quot.sound`; `sorryAx` means incomplete; names containing `_native` mean native evaluation was used; "Any other axiom means that a custom axiom was declared and used". The chapter distinguishes honest mistakes in proofs/meta-code from malicious code and names `comparator` and external checkers as the gold standard against adversarial proofs. This is the manual's own definition of what "verified" means for a Lean theorem: statement fidelity is outside the check.

**E09 — Zulip "Deadlocking the task system" (2023-07-23)** · https://leanprover-community.github.io/archive/stream/270676-lean4/topic/Deadlocking.20the.20task.20system.html · [S] · [FULL] · H1
Henrik Böving's example starts 8 `IO.asTask` jobs that each `Task.get` a nested task; the scheduler deadlocks because "Lean does spawn at max 8 (nproc on my machine) pthreads under the hood and it will forcibly run a Task to completion instead of switching to other tasks". Mac Malone: "waiting on a task from within another task… worthlessly consumes a thread in the thread pool and is a known pitfall with tasks in Lean". Compare E02, which now documents that `Task.get` grows the pool (older thread, newer documentation: see Conflicts C6).

**E10 — Zulip "structured concurrency" (2021-04-26)** · https://leanprover-community.github.io/archive/stream/270676-lean4/topic/structured.20concurrency.html · [S] · [FULL] · H1
Sebastian Ullrich: "Tasks are just standard futures available in virtually any stdlib, not structured concurrency." Mario Carneiro distinguishes pure tasks (terminated when the handle is dropped) from `asTask` tasks (outlive their caller).

**E11 — Zulip "Task code not running in parallel" (2024-02-08)** · https://leanprover-community.github.io/archive/stream/270676-lean4/topic/Task.20code.20not.20running.20in.20parallel.html · [S] · [FULL] · H1
User report of binary-splitting `Task.spawn` code that "doesn't seem to be parallelizing in practice"; asks for a thread-count flag. Practitioner usability evidence only.

---

## 3. Std.Do / Std.WP / mvcgen / vcgen: what it supports (sub-claims H1, H2, H4)

**V01 — Lean Reference chapter 17 "The vcgen tactic"** · https://lean-lang.org/doc/reference/latest/ (vcgen) · [P] · [FULL] (landing + syntax block; per-feature subpages were not read) · H1, H2, H4
Verbatim: "`vcgen` will break down a Hoare triple proof goal like ⦃P⦄ prog ⦃Q⦄ into verification conditions, provided that all functions used in prog have specifications registered with @[spec]."; "`vcgen` is experimental and subject to change."; "vcgen works on any program type Prog that carries a Std.WP.WP interpretation. A monad is one such program type. An inductive type of commands is another, once its weakest precondition is defined in terms of an operational semantics."; "The assertion lattice is any CompleteLattice, and the entailment is its order, so a verification condition is an ordinary Lean goal that grind can attack." The chapter requires `import Std.WP`, `Std.Tactic.Do` and opening `Lean.Order`. Not stated on the page: any built-in treatment of concurrency, scheduling or nondeterminism.

**V02 — lean4 source: WP instances (Std.Do and Std.WP)** · lean4 @ a2f1a642ab (2026-11-06) · [P] · [CODE] · H1, H4
`src/Std/Do/WP/Basic.lean` (125 lines) module doc: "This library comes with pre-defined instances for common monads and transformers such as `WP Id .pure` … `WP (StateT σ m) (.arg σ ps)` … `WP (ExceptT ε m) (.except ε ps)` … `WP (EStateM ε σ) (.except ε (.arg σ .pure))`". Instances declared at lines 77 (Id), 80 (StateT), 83 (ReaderT), 86 (ExceptT), 89 (OptionT), 92 (EStateM), 105 (StateM), 107 (ReaderM), 109 (Except), 111 (Option). The newer `src/Std/WP/Monad/Instances.lean` (241 lines) declares `wpInst` for Id (l.43), ExceptT (55), OptionT (93), StateT (129), ReaderT (156), Option (193), Except (208), EStateM (222). `grep -rE "IO|BaseIO|EIO|IO.RealWorld|Async|Task|nondet|concurr|interleav|probab" src/Std/Do src/Std/WP src/Std/Tactic/Do` finds no `IO`, `BaseIO`, `EIO`, `Async` or `Task` instance; the only hits for "nondet" are `NondetFun` in `Std/WP/Assertion.lean` (l.41–55, "total nondeterministic functions" used in the assertion lattice for termination measures) – a feature of assertions, not of program choice. Of the 76 files under `tests/` that mention `mvcgen`/`vcgen`, my regex found none declaring an `IO`, `BaseIO`, `EIO`, `Task`, `Async` or `IO.Ref` program (so no test demonstrates IO programs; whether `IO` programs elaborate through the `EStateM` instance because `IO` unfolds to `EIO ... = EStateM ...` was not tested: no Lean toolchain was run, by disk-budget decision). Lean `Task` is `structure Task (α) : Type u where pure :: get : α` (Init/Core.lean:624) and `IO.asTask` is an `opaque` (Init/System/IO.lean:245): the logical content of a task is only its eventual value.

**V03 — lean4 source: size/proof content of concurrency and data libraries** · lean4 @ a2f1a642ab · [P] · [CODE] · H1, H2
Counting lines mentioning `theorem|lemma`: `Std/Sync` 1 across 2,883 lines (Mutex, Channel, Semaphore, Barrier, Notify, RecursiveMutex, SharedMutex, Broadcast, CancellationToken…); `Std/Async` 0 across 3,132 lines (Basic, Select, Timer, TCP, UDP, DNS, Process, Signal…); `Std/Net` 0/262; `Std/Http` 25/13,726; `Std/Time` 12/11,174. For comparison: `Std/Data/HashMap` 1,402 / 8,337 lines; `DHashMap` 3,186 / 22,633; `TreeMap` 1,959 / 11,458; `DTreeMap` 5,125 / 36,778; `Iterators` 324 / 6,334; `Init/Data/BitVec` 1,328; `ByteArray` 96; `UInt` 2,353; `Array` 1,873; `List` 3,054; `Std/Tactic/BVDecide` 558 / 12,796 lines; `Std/Sat` 418 / 6,651. Reading: in the repository, the concurrency/async/network libraries are essentially specification-free; the data-structure and bit-vector libraries carry thousands of theorems.

**V04 — Release 4.35.0 (2026-10-12)** · https://lean-lang.org/doc/reference/latest/releases/v4.35.0/ · [P] · [FULL] (highlights, vcgen, lake check, removals; not read: full PR list) · H2, D
"The main new feature of Lean 4.35.0 is vcgen, a faster and more general successor of mvcgen." "Lean 4.35 ships vcgen, a verification condition generator that replaces mvcgen as a mostly drop-in successor; mvcgen is now deprecated." vcgen "builds on the Std.WP framework, whose weakest precondition interpretation applies to any program type rather than only to monads: besides do-programs in any monad, vcgen verifies deeply embedded languages, such as that of x64 assembler or WebAssembly". Assertions "can live in any complete lattice, which opens vcgen to assertion languages beyond predicates on the state, such as probabilistic logics and separation logic". Performance (author-reported): "On the AddSubCancel benchmark, where the program is a stateful loop body repeated 2200 times, vcgen needs 0.32 s where mvcgen needs 66 s, and the kernel checks the resulting proof in 0.53 s instead of 246 [s]. The running time of vcgen grows linearly with the program size, at about 0.14 ms per loop iteration." "vcgen is still marked experimental". Also: `lake check` and `lake comparator` (external-kernel replay, bubblewrap sandbox on Linux; bundled checkers lean4lean, nanoda, con-leche, con-ron, leanchecker-paranoid); "#14953 removes Lean.reduceBool, Lean.reduceNat, Lean.ofReduceBool, Lean.ofReduceNat and Lean.trustCompiler … deprecated since 2026-02-01 in favour of asserting native evaluations with axioms, which is what native_decide and bv_decide already do". Whether the benchmark generalises beyond that synthetic program is not stated.

**V05 — Release 4.31.0 (2026-06-13)** · https://lean-lang.org/doc/reference/latest/releases/v4.31.0/ · [P] · [FULL] (header, highlights; not read: complete PR list) · H2
305 changes (105 features, 102 fixes, 13 performance). Highlights: new `do` elaborator behind `backward.do.legacy false`; `repeat`/`while` loops "became verifiable (#13209)" via `whileM` so that `mvcgen`/`mvcgen'` can discharge loop bodies "given a termination measure"; "#13644 adds the experimental mvcgen' tactic, a from-scratch reimplementation of mvcgen on the new SymM-based symbolic-evaluation framework. It can outperform mvcgen by a factor of over 100x on some synthetic benchmarks and aspires to be feature-complete with it."; #12965 "generaliz[es] the assertion language for pre-/post-conditions of monadic Hoare triples from SPred to any CompleteLattice"; large transparency/defeq work ("making definitional-equality checking properly respect transparency") that lists migration steps for breaking proofs (see §8).

**V06 — Release 4.33.0 (2026-08-10)** · https://lean-lang.org/doc/reference/latest/releases/v4.33.0/ · [P] · [FULL] (highlights; not read: PR list) · H1, H2
208 changes. `Std.Http` grows "into a working server" (#12146 H1 pure state machine, #12151 async server) and "#13511 graduates the Async and Http modules out of Internal into Std"; `Float`/`Float32` become non-opaque with `Float.Model` "validated against the native implementation on cases from Berkeley TestFloat" (#14079, #14091); `grind` fixes include "bitvector literals that were not normalized to the form grind expects could be treated as two distinct values, in one case yielding a proof the kernel rejects (#14371…)"; `lia` gets its own `@[lia]` set; `liaSteps` bounds hard linear-integer search; tactic config evaluation "takes about 6.2% of the time it did before"; LLVM 19→22 gives "up to 5%" fewer instructions.

**V07 — Releases 4.25.0, 4.26.0, 4.27.0** · https://lean-lang.org/doc/reference/latest/releases/v4.25.0/ , …/v4.26.0/ , …/v4.27.0/ · [P] · [FULL] (headers and highlights; not read: PR lists) · H2
4.25 (2025-11-14): 398 changes; "invariant suggestions in mvcgen" (`mvcgen invariants?`), `grind` interactive mode (`grind => …`, anchors, `finish?` script generation), non-commutative ring normalisation in `grind`. 4.26 (2025-12-13): 264 changes; `grind_pattern` constraints, `#grind_lint` diagnostic that "creates an artificial goal and runs grind" per `@[grind]` theorem, Lake semver dependencies. 4.27 (2026-01-24): 372 changes incl. 25 performance; module system declared no longer experimental (#11637); "backward.* options are only temporary migration aids and may disappear without further notice after 6 months" (#11304); language-server memory reduced by ~1 GB in Mathlib (#11162); `funCC` default (function-valued congruence closure) with the note that traces/counterexamples/proof terms change and `grind +revert` restores old behaviour; BitVec in `grind ring`/`grind lia`.

**V08 — Releases 4.28–4.30, 4.32, 4.34 (targeted)** · https://lean-lang.org/doc/reference/latest/releases/v4.28.0/ … /v4.34.0/ · [P] · [FULL] (retrieved whole; read: lines matching mvcgen, vcgen, bv_decide, native_decide, axiom, kernel, GMP, HTTP, Async and the 4.34 kernel section; not read: remainder) · H2, D
4.28: `solverMode` for `bv_decide`; Async framework introduced; "mvcgen with tac fail[s] if tac fails on one of the VCs". 4.29 (2026-03-27): "#12217 implements RFC #12216: native computation (native_decide, bv_decide) is represented in the logic as one axiom per computation… #print axiom will no longer show Lean.trustCompiler, but rather the auto-generated names of these axioms (with, for example, ._native.bv_decide. in the name)"; less memory for LRAT checking. 4.30: core HTTP data types (Request, Response, Status, Version, Method, Headers, URI, streaming Body); `#print axioms` re-enabled under the module system. 4.32: `mvcgen' until $t`, consolidated syntax. 4.34 (2026-09-14): "focuses on the kernel: three soundness issues, found with AI adversarial testing… all reported by Daniel Selsam (OpenAI)": #14806 `is_def_eq` caching order-independent (union-find "made a query's answer depend on which queries had been asked before"), #14807 `is_prop` must require a sort, #14838 freeze objects whose 32-bit refcount overflows ("on machines with at least 18GB of free RAM this could be turned into a use-after-free… and extended into a proof of False"), #14833 require GMP ≥ 6.3.0, #14849 cap kernel Nat numerals at 128 MB; "[the bugs] matter when checking proofs that arrive from an untrusted source, not for a development that only elaborates its own code". Also `bv_decide` inside `sym =>`/grind state and "ported to the SymM preprocessor that makes it up to six times faster".

---

## 4. Automation: papers, numbers, limits (sub-claims H2, H4)

**A01 — "Interactive Bitvector Reasoning using Verified Bit-Blasting" (OOPSLA 2025, Böving et al.)** · https://dl.acm.org/doi/10.1145/3763167 · [P] · [FULL] (page returned truncated after Table 2; the SMT-LIB result figures and references were cut off and are not used) · H2, D
Claims: "the first end-to-end verified bitblaster and associated tactic for a dependently typed ITP that is scalable, trustworthy, and is backed by a unified bitvector library"; "relying only on a small trusted code base (TCB) and one axiom indicating trust in the Lean compiler" (`ofReduceBool`); "In practice, unfortunately, the LRAT proofs are too large and kernel reduction too slow, so we have to execute and trust the result of a Lean program". Library size: "156 definitions and abbreviations, 939 theorems… 7496 lines of code". Evaluation set-up: LLVM/Hacker's Delight on 128 GB RAM, Ryzen 9 7950X3D; SMT-LIB runs capped at 20 min and 8 GB; Lean nightly-2025-06-27. LLVM InstCombine: 4,556 tests give 7,978 bit-vector theorems; "Both bv_decide and Bitwuzla solve 7948 theorems, and both fail on 13 theorems with bv_decide failing to prove an additional 17"; "the geomean time to solve for bv_decide is 60.20ms"; rewriting-only theorems take "13ms geomean"; when SAT is needed "bit-blasting and SAT solving take 60% of the total solving time for InstCombine and 69%… for Hacker's Delight"; LRAT trimming+verification "19% (InstCombine) and 14% (Hacker's Delight)". SMT-LIB 2024 QF_BV benchmark set: 46,191 problems against Bitwuzla and CoqQFBV (results not read). Scope: fixed-width bit-vector logic only ("the SMT-LIB standard supports only formulas over fixed-width bitvectors"); no claim about protocols.

**A02 — "lean-smt" CAV 2025 preprint (arXiv 2505.15796)** · https://arxiv.org/html/2505.15796v1 · [P] · [FULL] (read: abstract, design, proof coverage, evaluation, conclusion, appendix tables) · H2
"lean-smt offers a smaller trusted core"; proof reconstruction covers "around 200 of these [cvc5] proof rules, which amounts to approximately 30%". Benchmarks: 5,000 Sledgehammer-generated SMT-LIB problems (Seventeen Provers) and 24,817 unsatisfiable SMT-COMP 2024 problems, 128 GB nodes. Table 1 (5,000 problems): cvc5+Lean-SMT solved&proved 2,868, checked 2,866, checked without holes 2,847; veriT+Sledgehammer 2,211/2,180/2,180; Duper 1,116/1,116/1,116. "lean-smt takes less than a second to replay proofs for 98% of the benchmarks, with the remaining 2% taking less than 5 seconds." SMT-COMP: cvc5 produced 21,595 proofs; "lean-smt successfully verified 15,271 proofs (71%), despite relying solely on the Lean kernel for soundness. Ethos verifies 98%". Limits stated: proofs "may contain holes"; future work "to additional SMT-LIB theories, such as bit-vectors, floats, and strings", beyond ~200 rules, higher-order logic via lean-auto. Repo README (see R-leansmt) confirms supported theories.

**A03 — "Proofs for finite-field arithmetic" (FMCAD 2026, Barbosa et al.)** · https://hanielbarbosa.com/papers/2026fmcad-ffproofs.pdf · [P] · [FULL] (read: abstract/introduction/evaluation table lines) · H2
"Out of 403 unsat QF_FF instances, CVC 5 with proof production solves 392, all verified by Carcara, while Lean-SMT reconstructs 247." Shows lean-smt reconstruction rate on a theory (finite fields, only prime order fields p^n with n=1 handled by SMT solvers) at 61% of the cvc5-solved instances. No protocol content.

**A04 — "Lean-auto" (Qian et al., arXiv 2505.14929)** · https://arxiv.org/pdf/2505.14929 · [P] · [FULL] (read: evaluation section 8, figures table) · H2
Set-up: user-declared theorems of Mathlib4 on Lean v4.15.0; 176,904 theorems, 27,762 excluded, so 149,142 problems; 64-core c5ad.16xlarge, 128 GB, 10 s per theorem; premise selection replaced by "all the theorems used in T's human proof" (ideal premise selection). Result: "Lean-auto + Duper… solves 36.6% problems in our benchmark set, which is 5.0% better than the best previous tool Aesop" (54,570 theorems, cumulative time 1,092.5 in the table); Lean-auto VBS 14.8% unique solves; overall VBS 53.2%; "Lean-auto is significantly slower than existing tools on solved problems". Scope: library maths theorems, not programs or protocols.

**A05 — "Duper" (ITP 2024)** · https://drops.dagstuhl.de/storage/00lipics/lipics-vol309-itp2024/LIPIcs.ITP.2024.10/LIPIcs.ITP.2024.10.pdf · [P] · [FULL] (read: abstract, evaluation tables) · H2
Superposition prover written in Lean that "generates proofs directly in a dependently typed axiomatic framework". Table 1 (original benchmark problems solved, out of 5,000 per set; 1,000 for the third column of each group): Duper(−) 1176 / 1086 / 64 and 1111 / 934 / 231; Metis 1195 / 1120 / 202; Vampire 1285 / 2521 / 262 and 1331 / 2341 / 459; Zipperposition 1277 / 2209 / 277 and 1314 / 2122 / 354 (column group headers not re-derived beyond table position). The paper reports Duper below Vampire/Zipperposition on these sets.

**A06 — grind: Lean Reference chapter 16 and the IJCAR 2026 system description** · https://lean-lang.org/doc/reference/latest/ (grind), https://link.springer.com/chapter/10.1007/978-3-032-32589-1_7 (PDF retrieved) · [P] · [FULL] · H2
Chapter: "The grind tactic uses techniques inspired by modern SMT solvers to automatically construct proofs… all proofs are by contradiction". Paper: grind has an arithmetic module, a CutSat-style `lia` ("operates over any type that admits an injection into Z via the ToInt typeclass"), and a linear-order solver. Evaluation section (verbatim): "grind was released in August 2025 and has been rapidly adopted. As of February 2026, Mathlib uses grind approximately 3 400 times across 2 400 theorems. Of these, around 1 200 were initially authored using grind, while the other 1 200 were refactored after grind became available. These refactors averaged a reduction of 28% in lines of code, with ∼50 theorems seeing proofs reduced by 10 lines or more." CSLib, the Prime Number Theorem project and Tao's Analysis I "each also uses grind around 500 times". Future work: premise selection for large libraries and scaling "when very large (>1000)" goals (sentence truncated in extraction). Gap: the paper contains no timing table; run-time numbers come only from release notes (V04–V08) and the NFM slides (S02).

**A07 — Lean Copilot (arXiv 2404.12534)** · https://arxiv.org/html/2404.12534 · [P] · [FULL] (read: abstract, evaluation) · H2
"When assisting humans, Lean Copilot requires only 2.08 manually-entered proof steps on average (3.86 required by aesop); when automating the theorem proving process, Lean Copilot automates 74.2% proof steps on average, 85% better than aesop (40.1%)." Domain: mathematics theorems.

**A08 — Verina (arXiv 2505.23135 v3)** · https://arxiv.org/html/2505.23135v3 · [P] · [FULL] (read: abstract, intro, results sentences) · H2, H3
189 programming challenges with descriptions, code, specifications, proofs and test suites in Lean. "The best model, OpenAI o3, achieves a 72.6% code correctness rate, 52.3% for specification soundness and completeness, and a mere 4.9% proof success rate (based on one trial per task)." "Among theorem-proving LLMs, the best model, Goedel Prover V2 32B, achieved an 11.2% proof success rate in one trial."

**A09 — Audit of CLEVER (arXiv 2605.23772 v2, Sosso, Arora, Spitters, 30 Oct 2026)** · https://arxiv.org/pdf/2605.23772v2 · [S] · [FULL] (read: abstract, introduction, prior-work paragraph) · H2, H3
CLEVER is a Lean 4 verifiable-code-generation benchmark of 161 problems. The original paper reported "an end-to-end ceiling of 1/161"; this audit with agentic stacks (Claude Opus 4.6, GPT-5.4, Kimi K2.5) "finds arguably erroneous reference specifications in 80 of 161 entries", reaches "up to 98.8%" valid specifications, "up to 92.3%" implementation certification and "up to 96.2% end-to-end success over entries with sound premises" (59 of 161 by CLEVER's own end-to-end criterion). Benchmarks are single small functional programs; not protocols.

---

## 5. Protocol-oriented Lean tooling (sub-claims H1, H2, H3)

**P01 — Veil CAV 2025 paper** · https://link.springer.com/chapter/10.1007/978-3-031-98682-6_2 (PDF) · [P] · [FULL] (read: intro, language, VC generation, evaluation, related/future work) · H1, H2
Veil is a Lean-embedded verifier for transition systems; "demonically non-deterministic values" via `*`; SMT back-ends cvc5 and Z3 through lean-auto, default "timeout of 5 s per SMT query". Evaluation on 16 case studies (9 IvyBench with manually added invariants, 2 Paxos from Padon et al., 5 others): "total 1704 non-empty, non-comment lines of Lean code (average 100 lines per file), 85 actions and 185 invariants"; run on M4 MacBook Pro, 32 GB, cvc5 1.2.1, Z3 4.14.0, Lean 4.16.0. Beyond-EPR: Veil verifies Paxos, Vertical Paxos, Suzuki-Kasami, Reliable Broadcast, whereas "Ivy… times out for Paxos and Vertical Paxos". Soundness: Veil's VC generator is proven sound against the semantics. Future work: invariant inference, "compose properties of individually verified protocols", and general program verifiers. A search for "liveness" in the paper body finds no use of the word (only inside a reference title), i.e., the paper evaluates safety (invariants) only.

**P02 — Veil Dafny'26 lessons paper** · https://verse-lab.org/papers/veil-dafny26.pdf · [P] · [FULL] (read: intro, features, performance section) · H2
"Veil supports proof reconstruction (at a 3-5x performance penalty) using Lean-SMT… external SMT solvers do not have to be trusted". Section 3.5 "Performance is a Huge Issue": "Veil is more than 10x slower than Ivy, the most closely related tool, although Veil is still reasonable, with verification of small specifications taking on the order of a few seconds, and our largest case study taking 160 seconds, compared to Ivy's 50 seconds… when the goals are easy, the overhead imposed by our verified translation logic becomes prohibitive".

**P03 — Loom, POPL 2026 ("Foundational Multi-Modal Program Verifiers")** · https://ilyasergey.net/assets/pdf/papers/loom-popl26.pdf · [P] · [FULL] (read: abstract, effects, non-determinism, Veil 2.0 and Velvet sections, related work; not read: appendices/proof details) · H1, H2, H3
"We provide a library of monad transformers for such semantics, encoding a variety of computational effects, including state, divergence, exceptions, and non-determinism." Verification "considers demonic non-determinism, effectively checking that every execution satisfies a desired property"; also angelic non-determinism for symbolic execution. Loom "implements the proof principles for partial correctness" and proves total correctness by combining with a separate termination proof. Veil 2.0 benchmark: "17 specifications of 15 different distributed protocols". Concurrency in Velvet (Dafny-style verifier for an imperative language): "we use a simple non-deterministic scheduler to randomly interleave those iterations, following a conventional way to encode concurrency in Dafny" (SpMSpV example). Related work: distinguishes itself from interaction trees, noting Loom "works directly with Lean monads, and allows one to verify and run Lean code using its native executable semantics", and uses Lean's `Gen` monad for a randomised protocol simulator (NOPaxos). I could not confirm in this read the earlier-recorded claim "Velvet faster than Dafny on 24/27" (see leads). So: nondeterminism and interleaving-by-scheduler are supported in Loom/Veil layers, not in Lean core `Std.Do`.

**P04 — Veil repository README (Veil 2.0)** · https://github.com/verse-lab/veil (README raw) · [P] · [CODE] · H1, H2
"Veil is a foundational framework for (1) specifying, (2) implementing, (3) model-checking, and (4) proving safety and liveness properties of state transition systems, with a focus on distributed protocols." Commands `#check_invariants`, `#model_check`, `#temporal_check`, `#simulate`, `#updr`; `prove_temporal_by` for liveness; Examples directories include AWS DNS race, Stellar Consensus, LivenessMC with "CAVA benchmarks". Repo: 495 Lean files, 21,760 LOC, 37 `sorry`-matches, 22 `native_decide`-matches (see table). This README post-dates the CAV paper (P01) and mentions liveness (see Conflicts C4).

**P05 — VCVio README** · https://github.com/Verified-zkEVM/VCVio · [P] · [CODE] · H1, H2
"Machine-Checked Cryptographic Proofs in Lean": `OracleComp`/`ProbComp` for oracle computations, `evalDist` measure-valued semantics, `simulateQ` operational semantics, "A program logic with relational (pRHL-style) and unary (Hoare-style) proof modes"; covers ML-DSA, ML-KEM, Falcon, SLH-DSA and "computational interpretations of interactive and UC-style systems". Repo size 437,887 LOC; its own `AGENTS.md` is referenced by the README's build instructions.

**P06 — leanda README (Igor Konnov)** · https://github.com/konnov/leanda · [S] (single-author experiments) · [CODE] · H1, H2
"This repository contains my experiments with Lean 4. The main purpose is to see how Lean's features match the features of TLA+." Contents: two-phase commit (functional, system, propositional specs, refinement proofs, inductive-invariant safety proofs); EPFD eventually perfect failure detector under partial synchrony ("strong completeness" checked, "strong accuracy is work in progress"); Ben-Or agreement; single-height Tendermint with "Lean proofs transferred from TLAPS by Codex GPT 5.6". 25 Lean files, 19,196 LOC, 1 `sorry`.

**P07 — tla-plus-lean-workflow (A. K. Keshavan)** · https://github.com/akkeshavan/tla-plus-lean-workflow · [S] · [CODE] · H1, H3
Claims "six distributed-systems case studies carried through a five-phase verification pipeline" (TLA+ model checking → Lean functional model → Lean propositional proofs via Veil/SMT → Rust implementation "Aeneas functional translation where applicable" → Solarkraft runtime monitoring). Table in README: 2PC phases 1–4, OpenRaft 1–3, Token Mutex 1–5, Raft election 1–5, Paxos 1–3, Lightning (abstract) 1–3; "The single remaining sorry stub (OpenRaft `next_preserves_VFG`, BecomeCandidate case)". Only two protocols reach the Rust phase with Aeneas extraction and only two reach runtime monitoring. Unreviewed, submitted paper; repo has 8 `sorry`-matches by my count versus 1 stated.

**P08 — lean-smt README** · https://github.com/ufmg-smite/lean-smt · [P] · [CODE] · H2
"`lean-smt` currently supports the theories of Uninterpreted Functions and Linear Integer/Real Arithmetic with quantifiers. Mathlib is required for Real Arithmetic. Support for the theory of Bitvectors is at an experimental stage." "cvc5's proofs may contain holes, returned as Lean goals." Described as "beta".

**P09 — scidonia/leanin: "lean-concurrency-today.md" and README** · https://github.com/scidonia/leanin · [S] · [CODE] · H1
README: "LeanIn is a concurrency runtime with a work-stealing scheduler for Lean 4… we can use techniques found in tokio and node". Doc claims, flagged `[R]`: "Two hard external constraints": "Lean gives no hook to replace Task's scheduler" and "there is no Task-level waker"; "The proof obligation is unclaimed ground. No machine-checked work-stealing deque, scheduler, or scheduling bound exists in Lean, and no formal semantics of Lean's IO/Task exists at all". AGENTS.md states "A build is not evidence that runtime behavior or the Lean source-to-IO correspondence is proved." AI-assisted flag per §1.

**P10 — Lean FRO NFM 2026 talk (Leo de Moura, slides)** · https://leodemoura.github.io/static/nfm2026/ · [P] · [FULL] · H2, H3, D
Author slides (verbatim lines): "zlib in Lean: A general-purpose AI converted zlib (a C compression library) to Lean, passed the test suite, and proved the code correct." Industrial uses: Cedar ("Lean model ~10× smaller than Rust", "~100M differential random tests nightly", "Release gate: no Cedar version ships unless model, proofs, and differential tests are current" — i.e., the Lean model is linked to the Rust production code by differential testing); SampCert; Waimea (Trainium compiler, "~200,000 lines of Lean code"); SymCrypt via "Charon → Aeneas → Lean" ("They verify the Rust code as written by software engineers. No special verification-friendly subset"; "The proof maintenance burden is real and unavoidable."); Anneal (Google, unsafe Rust); Veil ("16 distributed protocol case studies. All 16 verified. Ivy failed on 2. 87.5% verified in under 15 seconds"); Kraken (x64 model); Move borrow checker ("39,000 non-blank lines. 267 commits… 27 working days… using Claude"); "Signal Shot is a public moonshot to verify the Signal protocol and its Rust implementation using Lean" (an intent, not a result). Scaling slide: "Issue: Verification condition generation with proof assistants has not scaled so far. Adam Chlipala's group documented this for Rocq. The same story holds in Lean." grind slide: "4,000+ grind uses in Mathlib." The slides do not give a protocol-level verified result for Signal.

**P11 — Lean FRO Year 4 Part 1 roadmap (Sep 2026–Feb 2027)** · https://lean-lang.org/fro/roadmap/y4-1/ · [P] · [FULL] · H2, H3, D
"External checking must become a routine part of every Lean user's workflow, not an expert-only activity." `lake check`, `lake check --paranoid`, mode where "`lake build` skips the official kernel checks". Scalability: "We will continue working on scalability, both for Mathlib and for software verification… we will complete the new do notation, including first-class support for intrinsic verification, and deliver improvements to the verification-condition generator". Proof automation: "We will continue investing in grind and bv_decide… we will focus on quick failure". "AI-generated proofs are becoming a major source of grind and bv_decide usage." Projects: Tau Ceti, Hex ("we write the specifications, and AI agents implement both the executable code and the formal proofs"), Palomar. "Nerodia" Lean/Python FFI. The roadmap still names a "verification-condition generator" without naming mvcgen or vcgen.

**P12 — Postmortem for Kernel Soundness Bug #14576 (2026-08-01)** · https://leodemoura.github.io/blog/2026-8-1-postmortem-for-kernel-soundness-bug-14576/ · [P] · [FULL] · D
"A soundness bug in the Lean kernel (#14576)… exploits a bug in the kernel's handling of nested inductive types"; fixed one hour after the report (#14577). "The bug is only reachable through metaprogramming, by sending the inductive declaration to the kernel directly. The frontend checks the arguments and catches the ill-typed term." The independent kernel nanoda also accepted the original repository because of an unrelated bug; "checking with an independent kernel still works, since it required two distinct bugs in two implementations". "lean4lean… the proof of consistency does not cover inductive types yet". On trust: "The elaborator is untrusted by design. Soundness cannot depend on an untrusted component refusing to build a bad term."

**P13 — Postmortem for the Kernel Soundness Bug Hunt (2026-08-24)** · https://leodemoura.github.io/blog/2026-8-24-postmortem-for-the-kernel-soundness-bug-hunt/ · [P] · [FULL] · D
Hunt with OpenAI internal models (Daniel Selsam) ran 2026-07-30 to 2026-08-20. "lake build does not protect users from adversarial proofs that try, for example, to corrupt memory using Lean extensions. The gold standard is comparator and external checkers". Fixes listed: #14613, #14616, #14806, #14807/#14843, #14838 (refcount overflow), #14833 (Linux builds used GMP 6.1.2 "which contained a bug that could be exploited to construct a proof of False"), #14849. v4.33.1 released 2026-08-21 with fixes. New GitHub tags `soundness` and `runtime-soundness`. Planned `lake check`, `lean-inductive-models`.

---

## 6. Maintenance, compatibility and library coverage (sub-claims H2, H4)

**M01 — Breaking-change evidence in the release notes read above** · V05, V06, V07 · [P] · [FULL]
4.27: `backward.*` options "are only temporary migration aids and may disappear without further notice after 6 months". 4.31–4.33: stricter transparency regime; documented migration: "scope `set_option backward.defeqAttrib.useBackward true in` over the affected declaration, switch `simpa using` to `simpa using!`, mark the relevant constants `@[implicit_reducible]`, or add the now-required projections explicitly to simp/dsimp calls". 4.27: `grind` default change (`funCC`) alters "traces, counterexamples, and proof terms". 4.35: `Lean.reduceBool`/`trustCompiler` removed, `mvcgen` deprecated. 4.33: `Float` goes from opaque to modelled. These show that tactic-heavy proofs need migration work between releases; the notes give no quantified proof-breakage rate. The SymCrypt statement in P10 ("The proof maintenance burden is real and unavoidable. The fact that it stays manageable is the point.") is the only quantity-free author statement on maintenance that I read in full.

**Library coverage (counts, V03):** BitVec (1,328 theorem/lemma lines), ByteArray (96), UInt (2,353), HashMap/DHashMap/TreeMap/DTreeMap (≈11.7 k combined theorem lines) and Iterators (324) live in the Lean core repository at `a2f1a642ab`. Mathlib and Batteries were not inspected in this phase (see Open Questions).

---

## 7. How sources use "verify" and "check" (sub-claim D)

- Lean Reference "Validating a Lean Proof" (E08): a theorem is "verified" if the statement elaborates and the kernel accepts a proof using only the three standard axioms; faithfulness of the statement to an informal meaning is a separate, human, obligation.
- bv_decide paper (A01): "end-to-end verified" refers to the bit-blaster plus a certificate checker, with trust in "the Lean kernel and the Lean compiler".
- Veil papers (P01, P02): "verify" = prove a safety invariant of a transition system model; "foundational" = the VC generator is proven sound against the model's semantics; the connection to a running implementation is by model-checking/simulation (`#simulate`, NOPaxos runtime tests, P03) or manual modelling, not by extraction.
- Loom (P03): "foundational" means "verification tools that are proven sound in a general-purpose proof assistant with a small trusted core".
- Cedar (P10): the Lean model is proven correct; the production Rust is related to it by "~100M differential random tests nightly" – a test-based link.
- SymCrypt/Anneal (P10): Rust is translated into Lean by Aeneas, so proofs are about the translation of the Rust code.
- FRO roadmap/postmortems (P11–P13): "check" = replay with the official kernel plus independent kernels via `lake check`; "comparator" = challenge/solution statement equivalence.
- vcgen (V01, V04): "verification" = discharge VCs derived from Hoare triples over a program type with a WP interpretation.

---

## 8. Conflicts (not resolved)

**C1 — Size of the trusted base.** Side A: the bv_decide paper (A01) says the TCB is "the Lean kernel and the Lean compiler", "much smaller than trusting a full SMT solver"; the Reference (E06–E08) describes the kernel as small and the elaborator as untrusted. Side B: the 2026 postmortems (P12, P13) report soundness bugs in the kernel and runtime (nested-inductive check, `is_def_eq` cache, `is_prop`, refcount overflow, GMP 6.1.2) and say `lake build` "does not protect users from adversarial proofs"; V08 says these "matter when checking proofs that arrive from an untrusted source, not for a development that only elaborates its own code". Not resolved: whether the practical TCB for a given protocol proof includes the compiler/runtime depends on `native_decide`/`bv_decide` use (one axiom per computation since 4.29, V08).

**C2 — Scaling of VC generation.** Side A: FRO NFM slide (P10): "Verification condition generation with proof assistants has not scaled so far… The same story holds in Lean." Side B: release 4.35 (V04): vcgen 0.32 s vs mvcgen 66 s on a 2,200-iteration loop-body benchmark, linear growth; release 4.31 (V05): mvcgen' ">100x on some synthetic benchmarks". The benchmarks are synthetic, produced by the tool's authors; the slide's claim is about general scaling. Not resolved: no independent measurement on a non-synthetic program was found.

**C3 — Existence of a formal semantics for IO/Task.** Side A: leanin (P09, [S]): "no formal semantics of Lean's IO/Task exists at all [R]". Side B: Reference (E01): IO is a world-passing state monad; `Task` is a structure with a `get` field (V02, Init/Core.lean:624). Likely a difference in meaning: a logical model of the API versus a model of scheduling, fairness and memory behaviour. The leanin claim is marked as an unverified recollection `[R]` in the repository itself.

**C4 — Does Veil cover liveness?** Side A: the CAV 2025 paper (P01) evaluates safety invariants only (no occurrence of "liveness" in the body). Side B: Veil 2.0 README (P04) says "proving safety and liveness properties" and lists `#temporal_check` and `prove_temporal_by`. These are different versions (Lean 4.16 paper vs v4.35 repo). Not resolved: no published evaluation of Veil liveness (time, coverage) was read.

**C5 — Scope of vcgen versus what is implemented.** Side A: 4.35 notes (V04) say assertions "open vcgen to … probabilistic logics and separation logic" and programs can be "x64 assembler or WebAssembly". Side B: the instance list in the checked-out source (V02) has WP instances for Id, StateT, ReaderT, ExceptT, OptionT, EStateM, Option, Except only, with no IO/Task/Async instance and no test over such programs; nondeterminism and interleaving appear in Loom/Veil/Velvet (P03) rather than in `Std.Do`/`Std.WP`. The statements are about different levels (framework generality vs shipped instances); not resolved.

**C6 — Task blocking behaviour.** Side A: 2023 Zulip (E09): nested `Task.get` inside `IO.asTask` deadlocks when tasks ≥ cores. Side B: current Reference (E02/V02 docstring): "the maximum threadpool size is temporarily increased by one while waiting so as to ensure the process cannot be deadlocked by threadpool starvation". No post-2023 thread read confirming the fix; I did not run code.

**C7 — Status of mvcgen.** 4.35 (V04): "mvcgen is now deprecated". 4.31/4.32 (V05, V08): `mvcgen'` is the experimental reimplementation. NFM slide (P10) still lists "mvcgen — Lean's VC generator for monadic programs". Roadmap (P11) does not name either. Naming/lineage unresolved (mvcgen → mvcgen' → vcgen).

**C8 — LLM proof success for verified code.** Verina (A08): best proof success 4.9% (o3) / 11.2% (Goedel-Prover-V2 32B), one trial. CLEVER audit (A09): original ceiling 1/161, audit stacks up to 96.2% end-to-end on sound-premise entries. Different benchmarks, models, dates (2025 vs 2026) and success definitions; the audit's authors argue the benchmark is flawed in 80/161 entries.

**C9 — Veil performance.** FRO slide (P10): "87.5% verified in under 15 seconds", all 16 verified. Veil authors (P02): "more than 10x slower than Ivy", largest case 160 s vs Ivy 50 s, with proof reconstruction at a "3-5x performance penalty". Both can hold (different cut-offs); not resolved.

**C10 — tla-plus-lean-workflow proof completeness.** README claims "The single remaining sorry stub" (P07); the table count finds 8 occurrences of `sorry` (unclassified; may be comments). Not resolved without building.

---

## 9. Phase 1 gaps — status

| Gap from Phase 1 | Status after Phase 1b |
|---|---|
| Std.Do/mvcgen support for IO, concurrency, nondeterminism | Closed by source: instance list and absence of IO/Task/Async/program-nondeterminism support (V02). Not run: whether IO programs elaborate via `EStateM` (no toolchain run). |
| Formal semantics of IO/Task | Partly: IO is a world-token state monad (E01); Task is an opaque-ish structure with `get` (V02); scheduling/memory model not documented in pages read (E02). |
| Real numbers for bv_decide | Closed for LLVM/Hacker's Delight (A01); SMT-LIB result counts not read (page truncated); "up to six times faster" SymM figure is the release note's wording only (V08). |
| Real numbers for grind | Partial: Mathlib adoption and 28% LOC reduction (A06); no run-time table in the paper; NFM slide count 4,000+. |
| Real numbers for lean-smt | Closed (A02, A03). |
| Releases 4.25–4.27, 4.31, 4.33 | Read (V05–V07). |
| Liveness / real-time evidence | Liveness: Veil README only (P04) plus leanda EPFD partial (P06). Real-time: only `IO.sleep`/`monoMsNow` primitives (E04). No timed-system tooling found. |
| Mathlib/Batteries coverage | Not done (see Open Questions). |
| LLM-prover figures for DeepSeek-Prover, Kimina, AlphaProof | Not done; moved to leads. |

---

## 10. Open Questions

1. Does a `vcgen`/`mvcgen` proof about a program in `IO` (with `IO.Ref`, `IO.asTask`) exist anywhere public? Static grep of 76 test files says no; no toolchain was run.
2. What is the documented memory model and scheduling guarantee of `Task`/`Async`? The Reference pages read give an API, not a semantics (E02, E03).
3. Is there a published Lean framework with a non-trivial (fairness-bearing) liveness proof for a protocol? Veil README claims `prove_temporal_by` (P04); leanda EPFD "strong accuracy" is unfinished (P06). Neither examined for proof size.
4. Can a Lean spec be linked to non-Rust production code (C, Go, TypeScript) with a stated guarantee? Evidence read: Rust via Aeneas (P10), differential testing for Cedar (P10); no C/Go path read.
5. What is the cost of proof maintenance across Lean versions for a protocol-sized development (Veil moved from Lean 4.16 to 4.35 between its paper and README)? No quantified data read.
6. Do independent re-measurements of vcgen/SymM/bv_decide-in-grind exist outside the FRO? None found.
7. Mathlib/Batteries coverage for finite maps and bitvectors, and the state of `Std` collections beyond counts (V03), were not inspected.
8. Signal Shot (P10) is stated as a "moonshot"; its deliverables and any result were not found.
9. How do the sources define "verify a protocol": safety invariants of a model (Veil), computational security (VCVio), or implementation correctness (Aeneas)? Section 7 lists usage; no source discusses the hypothesis' phrase directly.

---

## 11. Unverified leads (excluded from evidence)

These were found in Phase 1 or this phase but not read in full (snippet-only, extractor-read, truncated, or not yet fetched). Nothing above relies on them.

1. lean-lang.org homepage "absolute correctness" wording (extractor-read in Phase 1).
2. Velvet "faster than Dafny on 24/27" (Phase 1 note; not found in the Loom text sections read).
3. SymM "MetaM 2073 ms at n=100 vs 16 ms" (extractor figure; not present in the NFM slide text read).
4. bv_decide SMT-LIB solved counts (41,825 / 99.3x etc., snippet only; the fetched page was truncated).
5. Releases 4.11, 4.12, 4.16, 4.19, 4.24 (cited in Phase 1 via extractor; not re-read).
6. Release 4.32.2 and 4.33.1 patch pages.
7. Release v4.36.0-rc2 page.
8. Leo de Moura SVIL26 and FLoC26 slides (https://leodemoura.github.io/static/floc26/ , https://sebasti.a.nullri.ch/files/floc2026.pdf).
9. Kimina-Prover (arXiv 2504.11354).
10. DeepSeek-Prover-V2 (arXiv 2504.21801).
11. Goedel-Prover-V2 (arXiv 2508.03613; only the Verina quote of its 11.2% was read).
12. AlphaProof (Nature).
13. querysmt (arXiv 2601.14495).
14. SymCrypt/Aeneas paper (arXiv 2609.15648).
15. GitHub issue lean4#1280 "Additional concurrency primitives" (opened 2022-07-04; only the opening text was read; outcome not read).
16. Zulip "FRO's new verifier" thread (25 KB; only questions and Henrik Böving's "comparing a challenge and a solution file" answer read — noted for context only).
17. eigenius toolchain-upgrade note (https://github.com/eigenius/eigenius/blob/main/docs/notes/lean-toolchain-upgrade.md).
18. Oraclizer blog (https://research.oraclizer.io/why-were-starting-formal-verification-applying-our-own-standard/).
19. mizchi formal-methods-playground verification-tools guide.
20. Quint "formal verification tools" guide (https://quint.sh/guides/formal-verification-tools).
21. TLAPS-Bench.
22. RELEASES.md compatibility policy (old snapshot only).
23. Mathlib/Batteries documentation and benchmarks; Aesop, omega, simp documentation pages.
24. "crush" arXiv report on grind VC counts (85.0% of 754 VCs) and "learned interventions" report (Phase 1 extractor numbers).
