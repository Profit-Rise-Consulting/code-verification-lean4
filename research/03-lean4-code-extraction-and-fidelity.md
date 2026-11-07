# 03 – Lean 4 code extraction and fidelity (Phase 1b)

**Header counts:** [FULL] entries: 23 | [CODE] entries: 18 (several [CODE] entries also include full reading of the repo's own docs) | Unverified leads (excluded from evidence): 19. Depth tags are per entry. Neutral record; no verdict on the hypothesis.

Access date for every URL: **2026-11-07**. Tags: **[P]** primary, **[S]** secondary. Sub-claim IDs (H1–H5, D) follow `research/CONVENTIONS.md`. This file bears mainly on H3 (implementation link), H4 (generality), H5 (inference) and D (definitions); it touches H1/H2 only where a source says so.

Scope split: separate dossiers cover Cedar, SampCert, LNSym, Veil/Lentil/Velvet/Loom, Aeneas/SymCrypt, hax and others in `research/references/`. Entries here for Aeneas, SymCrypt, hax and LNSym are limited to facts needed for the code-extraction picture; see those dossiers for depth.

---

## 0. Method and provenance notes

- Web pages and papers were downloaded and read in full as text (`pandoc` for HTML, `pdftotext` for PDF). Long pages (Lean release notes, the SymCrypt paper, the NFM slides, the Charon paper) were read at the sections quoted; this is stated per entry.
- Repositories were shallow-cloned into `/tmp` (`--depth 1 --filter=blob:limit=1m`). Counts below come from a crude `grep` over `*.lean` files (excluding `.lake` and `.git`). They count textual matches, including comments and doc strings (for example `admit` in `lean-surfaces` matches the C++ semantics vocabulary, not the tactic). They are indicative only. Where a repo documents its own counts, those are quoted separately.
- Disclosure: during Phase 1b a few GitHub REST API calls were made early (repository archive flag and pushed date for about 18 repos; one README each for Perennial, julia-in-lean and Goose; existence of two Go issues). This contravenes CONVENTIONS rule 3. None of those results is used as evidence here without a second read: the Clear archive status was re-read from the GitHub web page, the Go issues from the web pages, and the Perennial and julia-in-lean READMEs from cloned copies. The Goose README read through the API is not used.
- AI-generated or AI-assisted repositories are flagged by what was observed: presence of `AGENTS.md`, `CLAUDE.md`, `.claude/` or `.agents/` files, and README self-statements. This is a determination from repo files, not from commit history (shallow clones have one commit).
- A correction to the Phase 1 version of this file: the Phase 1 text described lean-zip as a case with `@[extern]` zlib opaques, taken from a secondary summary. The README read in Phase 1b describes a pure-Lean DEFLATE implementation (entry E37).

---

## 1. Lean core: trusted base, compiler, FFI

### E01 Lean reference manual, "Axioms" — https://lean-lang.org/doc/reference/latest/Axioms/ — [P] [FULL] — D, H3
- States that axioms are "postulated constants". "Any proof that relies on an axiom can be trusted only to the extent that the axiom is both true and consistent with the other axioms used." Lean "cannot check whether new axioms are consistent."
- On native evaluation: "The `native_decide` tactic creates a bespoke axiom for each invocation", so each one can be audited. Native-evaluation axioms record proofs depending on "the correctness of the entire compiler, and not just on the much smaller `kernel`."
- Lists `Classical.choice`, `propext`, `Quot.sound` as the standard axioms and `sorryAx` as the placeholder that "can be used to prove anything".
- Does not itself define a "trusted base".

### E02 Lean reference manual, "Validating a Lean Proof" — https://lean-lang.org/doc/reference/latest/ValidatingProofs/ — [P] [FULL] — D, H3
- The page's remaining-issues discussion states that the `Lean.trustCompiler` machinery was removed from Lean in version 4.35.0, and that external checkers cannot check proofs that use native evaluation (verified by text search of the downloaded page; wording paraphrased here).
- Relevance: the official description of what a Lean proof does and does not establish about compiled code.

### E03 Lean reference manual, "Foreign Function Interface" — https://lean-lang.org/doc/reference/latest/Run-Time-Code/Foreign-Function-Interface/ — [P] [FULL] — H3, H4
- "The current interface was designed for internal use in Lean and should be considered unstable. It will be refined and extended in the future."
- "Lean offers efficient interoperability with any language that supports the C ABI. This support is, however, currently limited to transferring Lean data types; in particular, it is not yet possible to pass or return compound data structures such as C structs by value from or to Lean."
- Attributes: `extern` "Binds a Lean declaration to the specified external symbol"; `export` "Exports a Lean constant with the unmangled symbol name sym".
- Borrow annotation: "From the perspective of Lean's type system, this annotation has no effect."
- Does not state any correctness property of the foreign code, nor of Lean's C emission.

### E04 Lean reference manual, "Reference Counting" — https://lean-lang.org/doc/reference/latest/Run-Time-Code/Reference-Counting/ — [P] [FULL] — D, H3
- Describes Lean's runtime memory model: reference counting made explicit in the compiler's IR. Read for the compiler-pipeline description only. States no correctness theorem for the compiler.

### E05 Lean release notes 4.22.0, 4.30.0, 4.31.0, 4.35.0 — https://lean-lang.org/doc/reference/latest/releases/v4.22.0/ ; .../4.36.0-rc1/releases/v4.30.0/ ; .../latest/releases/v4.31.0/ ; .../stable/releases/v4.35.0/ — [P] [FULL] (relevant entries) — H3, H4, D
- 4.35.0 headline: "The main new feature of Lean 4.35.0 is vcgen, a faster and more general successor of mvcgen. It also adds lake check and lake comparator, which check a project with Lean's kernel and, if requested, with several independent kernels that now come with the toolchain."
- 4.35 vcgen: "vcgen builds on the Std.WP framework, whose weakest precondition interpretation applies to any program type rather than only to monads: besides do-programs in any monad, vcgen verifies deeply embedded languages, such as that of x64 assembler or WebAssembly for which a notion of weakest precondition is definable." Performance claim: on the AddSubCancel benchmark "vcgen needs 0.32 s where mvcgen needs 66 s".
- 4.31.0: assertion language generalised to any `CompleteLattice`; experimental `mvcgen'` "can outperform `mvcgen` by a factor of over 100x on some synthetic benchmarks".
- 4.30.0 (read: LCNF entries): the release notes describe completing the new LCNF compiler backend and porting the C emission pass from IR to LCNF ("marking the final step of the IR/LCNF conversion"); see E11 for the older C++ pipeline.
- 4.22.0: introduces `Std.Do.Triple` and `mvcgen`.

### E06 Lean reference manual "The mvcgen tactic" and tutorial "Verifying Imperative Programs Using mvcgen" — https://lean-lang.org/doc/reference/latest/The--mvcgen--tactic/ ; .../Overview/ ; https://lean-lang.org/doc/tutorials/latest/mvcgen/ — [P] [FULL] — H3, D
- Banner: "The mvcgen tactic and its supporting libraries have been superseded by vcgen, which supports a wider variety of proofs and performs better. New code should use vcgen, and existing code should migrate."
- "The mvcgen tactic implements a monadic verification condition generator: It breaks down a goal involving a program written using Lean's imperative do notation into a number of smaller verification conditions (VCs) that are sufficient to prove the goal."
- Workflow: programs are given a predicate-transformer (weakest-precondition) semantics; "loops require a specified loop invariant"; leftover VCs are proved by a proof mode or ordinary tactics.
- Tutorial spec form (verbatim): `⦃⌜True⌝⦄ mkFreshN n ⦃⇓ r => ⌜r.Nodup⌝⦄` and `@[spec] theorem mkFresh_spec (c : Nat) : ⦃fun state => ⌜state.counter = c⌝⦄ mkFresh ⦃⇓ r state => ⌜r = c ∧ c < state.counter⌝⦄`.
- Scope: the verified program is a Lean program (shallow embedding in monads), or, per E05, a deeply embedded language for which a weakest precondition is defined. The documentation does not address code written in other languages.

### E07 Lean4Lean (paper) — https://arxiv.org/html/2403.14064 — [P] [FULL] — D, H3
- "an external verifier for Lean 4, written in Lean. The verifier itself is more than a prototype: it is fully capable of checking the entirety of Lean (the Lean compiler package), Batteries (the extended standard library), and Mathlib (the Lean mathematics library)." Reimplementation "closely following the kernel" (conclusion).
- Verification scope (section 4.1): "Currently, only a few functions from the typechecker have been verified, like inferLambda. However, in the course of performing the verification we have already run into some interesting issues, and at least one soundness bug (which has since been fixed in response to our report)."
- On `reduceBool` (section 3): IR "can also call external functions from C, and with implemented_by one can completely decouple the compiler behavior from the kernel's version of the implementation, so the general feature is simply 'unsound by design.' Given that Mathlib and many other mathematics projects avoid reduceBool completely for these reasons, we do not lose much by not supporting it."
- Conclusion states the long-term aim: "a fully verified, self-hosted Lean kernel". The paper does not claim verification of the Lean compiler.

### E08 Lean4Lean repository — https://github.com/digama0/lean4lean — [P] [CODE] — D
- SHA `67f63b0c8189`, last commit 2026-10-10, toolchain `leanprover/lean4:v4.33.0-rc2`. Lean files 112, LOC 49,412. Crude counts: sorry 131, admit 2, axiom 73, native_decide 0, implemented_by 14, extern 0, opaque 36, partial 48.
- README: "This is an implementation of the Lean 4 kernel written in (mostly) pure Lean 4. It is derived directly from the C++ kernel implementation, and as such likely shares some implementation bugs with it (it's not really an independent implementation)". Also "The project also houses some metatheory regarding the Lean system".
- Not flagged AI-generated (no AGENTS/CLAUDE files).

### E09 "Type Checking in Lean 4", trust chapter — https://ammkrn.github.io/type_checking_in_lean4/trust/trust.html — [S] [FULL] — D, H3
- "Trusting the correctness of compiled Lean programs requires trust in Lean's compiler, which is separate from the kernel and is not part of Lean's core logic."
- Lists, for compiled executables: "You trust that any compiler overrides (extern / implemented_by) do not violate the lean logic (i.e. the model matches the implementation)"; "You trust the lean compiler (which lowered the lean code to C) to preserve the semantics of the program"; "You trust clang / LLVM to convert the C program into an executable with the same semantics".

### E10 Lean Zulip archive, "soundness bug: native_decide leakage" — https://leanprover-community.github.io/archive/stream/270676-lean4/topic/soundness.20bug.3A.20native_decide.20leakage.html — [S] [FULL] — D
- Oct 2023 thread. Mario Carneiro: "You don't need the ofReduceBool axiom for weirdness from the real world to affect the logical system", with a program using `IO.getRandomBytes` and `Lean.reduceBool` that proves `False` with no listed axioms (non-deterministic, "only works with 1/4 probability"). Proposed fix: make the dependency show up in `#print axioms`; Scott Morrison cites lean4#2654 adding an axiom that `Lean.reduceBool` and `Lean.reduceNat` depend on. Dated 2023; later behaviour is in E01/E02/E05.

### E11 Lean Zulip archive, "The state of the code generator" — https://leanprover-community.github.io/archive/stream/270676-lean4/topic/The.20state.20of.20the.20code.20generator.html — [S] [FULL] — H3
- Thread from 2023 about the old C++ code generator versus the in-progress Lean-implemented one, C backend, and an LLVM backend. Dated; superseded in part by E05 (4.30.0).

### E12 Lean FRO talk slides "Lean 4: A Unified Platform" (NFM 2026) — https://leodemoura.github.io/static/nfm2026/ — [P] [FULL, read at sections cited] — H3, D
- Slides cited in the Phase 1 version for: SampCert "Custom code extractor: the deployed artifact does not go through the Lean compiler. TCB includes the extractor, not the compiler."; Cedar differential random testing counts; a Move borrow-checker formalisation "39,000 lines of mechanized metatheory in 27 working days, using Claude as an AI coding assistant". Quotes re-verified against the page text (section 9). Slides are author summaries without detail.

---

## 2. Rust to Lean

### E13 Charon (paper) — https://arxiv.org/html/2410.18042v2 — [P] [FULL, sections 1–5] — H3, H4
- Charon is "implemented as a custom driver for the rustc compiler" (projects page, E15) producing MIR-derived ULLBC and then LLBC.
- Paper statement on Aeneas: "Aeneas comes with restrictions, notably, the programmer cannot rely on runtime-checked aliasing (RefCell) or concurrency."
- "some less-central features such as generic associated types, dyn Trait, or trait aliases are not yet fully supported."
- "a declaration that cannot be translated is marked as missing and the rest of the crate is translated as usual".
- Numbers read: Charon is "18kLoC" plus an 11 kLoC frontend-exporter crate; evaluation targets include a safe-Rust HACL* port ("about 84kLoC") and an ML-KEM implementation from libcrux (12 kLoC). "On the evaluated crates, translation from Charon to LLBC takes on average 80 s". The paper was not read as claiming Charon itself is formally verified; no such claim was found.

### E14 Aeneas README — https://github.com/AeneasVerif/aeneas — [P] [FULL] — H3, H4
- Aeneas is "a verification toolchain for Rust programs" and "relies on a translation from Rust's MIR internal language to a pure lambda calculus". Backends "F*, Coq, HOL4 and LEAN"; "Our most mature backends are Lean and HOL4".
- "Aeneas currently functionalizes a subset of safe Rust, and we are in the process of extending the functional model with reasoning based on separation logic to support unsafe code and concurrent code."
- Loop limitation: "`return` inside nested loops, or `break`/`continue` to outer loops are not supported yet". Also instantiating a generic type with a mutable reference "is not supported yet".
- Formal status, verbatim: "The translation has been formalized and published at ICFP2022 ... We also have a proof that the symbolic execution performed by Aeneas during its translation correctly implements a borrow checker, and published it at ICFP 2024".

### E15 Aeneas/Charon projects page — https://aeneasverif.github.io/projects/ — [P] [FULL] — H3, H4
- Charon pipeline MIR → ULLBC → LLBC. Aeneas "currently targets a safe, sequential Rust code."
- Scylla (C → safe Rust): "a tool for translating highly regular C code to safe Rust." "As of July 2025, Scylla is able to translate parts of the HACL* and SymCrypt cryptographic libraries, as well as the CBOR binary parser from EverParse, and the core compression algorithm of bzip2, all of them requiring different levels of manual C source rewritings and annotations."
- Eurydice (Rust → C) listed; see E19.

### E16 "Scaling Verification ... SymCrypt" (Microsoft Research) — https://arxiv.org/html/2609.15648v1 — [P] [FULL, abstract, trust section, sections 5–8] — H3, H4, D
- Scope: Rust implementations in SymCrypt (SHA-3, ML-KEM, and others) verified in Lean with Aeneas: proofs of "safety, panic-freedom, and functional correctness". Numbers (abstract and §8): 237 KLOC of Lean for 16.7 KLOC of Rust; proofs in part agent-generated.
- Trust list (verbatim items from the paper's trust section): "we trust the Rust toolchain, notably its compiler and its platform-specific back-ends for x86-64 (including SSE2/AVX2) and AArch64 (including NEON)"; trust "that Charon correctly extracts MIR into LLBC"; "that Aeneas correctly translates LLBC into Lean"; "the soundness of the Lean kernel that checks all proofs in our development"; "the soundness of the compiled native_decide and bv_decide tactics"; "the soundness of the Aeneas axiomatization for some functions of the Rust standard library".
- Specification: "we trust the Lean formal specification faithfully captures the standard". The paper states it verifies functional correctness only, not leakage resistance.
- Unsupported Rust (unsafe, raw pointers, external code) is handled with `verify::opaque` and hand-written models or axioms; hardware intrinsics are Rust shims.
- Aeneas dossier owns further detail.

### E17 hax (Cryspen) — three pages, all [P] [FULL]
- (a) Manual intro https://hax.cryspen.com/manual/ — "hax is a tool for high assurance translations of a large subset of Rust into formal languages such as Lean, F* or Rocq." Lists backends `lean`, `legacy-lean`, `fstar`, `coq`, `pro-verif`, `ssprove`, `easycrypt`. Does not list unsupported features and gives no translation-correctness statement.
- (b) Lean quick start https://hax.cryspen.com/manual/lean/quick_start/ — "hax doesn't support all Rust constructs, e.g, `unsafe` code or interior mutability."
- (c) hax 0.4 release post https://hax.cryspen.com/blog/2026/09/29/hax-04-easier-to-install-and-configure-and-a-new-lean-backend/ — "The new Lean backend does not use the hax engine. Instead, the `cargo-hax` command line tool drives Charon and Aeneas". `hax_lib::requires`/`ensures` become pre/postconditions; generated theorems begin with `sorry`; finished example uses `hax_mvcgen` and `grind`. Version combinations other than the pinned ones "are untested". Tags: H3, H4.
- hax dossier owns further detail.

### E18 "Rust-to-Lean pipeline with AI provers" (third-party evaluation) — https://arxiv.org/html/2605.30106 — [S] [FULL] — H3, H4
- Statements: "The supported subset of safe Rust that Aeneas and Hax can extract to pure Lean is finite." Generic functions with trait bounds "cannot be extracted directly: Aeneas needs concrete monomorphised types." External-crate calls are not extractable because Charon does not see MIR of crates beyond the workspace. Rust `while` loops become recursive functions needing termination measures. Says the Lean backend of hax is "under active development" and cites "independent analysis" of semantic gaps (reference not identified). No extraction success rate given.

### E19 Eurydice README — https://github.com/AeneasVerif/eurydice — [P] [FULL] — H3
- Rust → C compiler. Informal guarantee quoted: if the Rust program terminates without panicking, the compiled C code computes the same result without undefined behaviour; no guarantee if the Rust program panics. Not a Lean tool; recorded as part of the Aeneas ecosystem.

---

## 3. Machine-code, bytecode and binary-level routes

### E20 LNSym README — https://github.com/leanprover/LNSym (raw README.md) — [P] [FULL] — H3, D
- Title: "LNSym: Native Code Symbolic Simulator in Lean". Opening line: "LNSym is a symbolic simulator for Armv8 machine-code programs." The README read in full contains build prerequisites, repository layout and test targets (including a `cosim` conformance-testing target); it contains no list of supported instructions and no limitation statement. LNSym dossier owns detail.

### E21 Amazon Science blog "How the Lean language brings math to coding and coding to math" — https://www.amazon.science/blog/how-the-lean-language-brings-math-to-coding-and-coding-to-math — [P] [FULL] — H3, D
- "LNSym uses Lean as a specification language to model the Arm instruction semantics and cryptographic protocols and as a theorem prover for reasoning about these artifacts. Since Lean programs are executable, the specifications achieve a high degree of trust through thorough conformance testing." Also describes LNSym as "currently under development, with a focus on enabling automated reasoning of cryptographic machine-code programs."

### E22 LeanCIL (Nethermind) — https://github.com/NethermindEth/LeanCIL — [P] [CODE + docs] — H3, H4, D
- SHA `f96b016a7c6b`, last commit 2026-11-05, toolchain `leanprover/lean4:v4.35.0`, licence file present (Demerzel Solutions Limited copyright; GitHub reports NOASSERTION). Lean files 495, LOC 80,084. Crude counts: sorry 5, admit 5, axiom 1, native_decide 0, implemented_by 3, extern 0, opaque 72, partial 29. `AGENTS.md` present (AI-agent-assisted development indicated by file).
- README: "LeanCIL uses Lean 4 to prove properties of compiled .NET code." Supported: integer arithmetic, selected control flow and managed calls, reference-free structs, supported integer vectors over supplied storage. "Unsupported instructions and signatures are rejected explicitly." Outside subset: general class allocation and virtual dispatch, ordinary recursive calls, exception handlers, floating point, arbitrary external calls. "LeanCIL does not claim complete CLR or JIT verification."
- `docs/SOUNDNESS.md` table of trusted components (verbatim): "CIL and memory semantics | The definitions correspond to the admitted CLR behavior. This correspondence is reviewed, not proved against a CLR implementation." "Metadata decoding and extraction | Mono.Cecil and the extractor faithfully decode and serialize the assembly". "Runtime helpers and intrinsics | Each admitted external method implements the modeled behavior on the selected runtime and architecture." Specification baseline: ECMA-335 (2012) plus tracked augments.
- Source languages: any language compiling to CIL (C# targeted); the verified object is the compiled assembly, not the source.

### E23 EVMYulLean — https://github.com/NethermindEth/EVMYulLean — [P] [CODE] — H3
- SHA `047f63070309`, last commit 2025-09-24, toolchain `leanprover/lean4:v4.22.0`. Lean files 66, LOC 11,339. Crude counts: sorry 1, axiom 0, implemented_by 1, extern 4, opaque 4. Apache-2.0 (as reported on the repo). README: "formal model of the EVM and Yul in Lean 4" and "Everything here is work in progress". The 99.99% Cancun-test figure from Phase 1 came from a Nethermind web page and is not re-verified here (lead L05).

### E24 "Ship your interpreter" (BasisResearch) — https://github.com/BasisResearch/ship-your-interpreter — [P] [CODE + README] — H3, D
- SHA `fb07094cff91`, last commit 2026-10-01, toolchain `leanprover/lean4:v4.34.0`, no root licence file detected. Lean files 1,113, LOC 421,771. Crude counts: sorry 1, axiom 75, native_decide 0, partial 93, unsafe 11. Vendored Sail-generated RISC-V model.
- README: "This Lean 4 project verifies `c/while-riscv-htif.elf`, a WHILE-language interpreter written in C and compiled to bare-metal RV64. The proof relates an inductive big-step semantics of WHILE to the binary's execution in the Sail-generated RISC-V model."
- Verbatim theorem (README): `theorem endToEnd_refinement : ∀ p c, Loaded interpRunLayout p (fillZero c) → (∀ out, BigStep p out ↔ Halts c out 0) ∧ (Diverges c → ¬ ∃ out, BigStep p out)`.
- Stated assumptions: the only hypothesis is `Loaded ...` (initial machine state). "Two links between those witnesses and the ELF file are not checked by the kernel: that loading the ELF gives the witness's initial memory, and that the traced boot is the machine's. The scripts that checked them, and the generator that produced the witness files, are not in this repository."
- Demonstrates a C-to-binary-to-Lean chain for one interpreter; the C compiler is outside the proof because the artifact is the ELF.

---

## 4. Translators and models of other source languages

### E25 Clear (Nethermind) — https://github.com/NethermindEth/Clear — [P] [CODE] — H3
- SHA `6ef2e7eda010`, HEAD commit date 2025-08-07, toolchain `leanprover/lean4:v4.9.1`. Lean files 42, LOC 6,227. Crude counts: sorry 0, axiom 0, partial 8, unsafe 81 (mostly doc/comment matches).
- GitHub page banner (read via web): "This repository was archived by the owner on Sep 9, 2026. It is now read-only." README: "Prove anything* about Yul programs." Two parts: "A Lean framework with a Yul model" and "A verification condition generator."
- Nethermind blog (lead L06, not re-read). Source-language route: Solidity compiled to Yul then Yul embedded in Lean.

### E26 Verity — https://github.com/lfglabs-dev/verity — [P] [CODE + docs] — H3, H4, D
- SHA `16e84d8d1182`, last commit 2026-11-07, toolchain `leanprover/lean4:v4.31.0`, MIT. Lean files 773, LOC 362,938. Crude counts over all `.lean`: sorry 29, admit 19, axiom 12, native_decide 579, implemented_by 24, extern 12, opaque 44. `AGENTS.md` and `CLAUDE.md` present (AI-agent-assisted indicated).
- README, verbatim: "The compiler proof stack carries 0 axioms and no `sorry`. Yul → bytecode is done by pinned `solc` and is trusted. Verification is scoped: the proved fragment and everything trusted are listed in the trust documents below." Layers: Layer 2 (CompilationModel → IR) "a generic whole-contract theorem covers the supported fragment with zero axioms"; Layer 3 (IR → Yul) "every statement form is proved equivalent. The dispatch bridge is an explicit theorem hypothesis, not an axiom."
- Docs: `docs/AXIOMS.md` registry; `docs/TRUST_ASSUMPTIONS.md` lists trusted components (Lean kernel, `solc`, EVM spec, deployment toolchain, named external assumptions). Counts above include non-proof directories; the repo's own claim is limited to the compiler proof stack.
- Solidity import path exists ("Write a contract in an embedded DSL (or import it from Solidity)" per README; import docs describe a whitelist-based smoke theorem).

### E27 LemmaScript — https://github.com/midspiral/LemmaScript — [P] [CODE + docs] — H3, H4, D
- SHA `385cfbdf4ffb`, last commit 2026-11-04, toolchain `leanprover/lean4:v4.34.0`, MIT. Lean files 94, LOC 3,755. Crude counts: sorry 0, axiom 0. `AGENTS.md` present (AI-agent-assisted indicated).
- README: TypeScript with `//@` annotations compiled to Dafny or Lean ("with Velvet 2"); marked "Tech Preview". `docs/SUBSET.md` lists the supported TypeScript subset; `docs/SUBSET.md`: "Both Dafny and Lean are supported, but Dafny currently understands a wider set of library methods (most string/array helpers below are Dafny-only)... When in doubt, target Dafny." Per-backend columns are in SPEC.md. `DESIGN.md` states the translation is validated by inspection of the code generator rather than proved.
- The Lean backend generates definitions; specs and proofs are hand-written files.

### E28 Thales (TypeScript to Lean) — https://github.com/jessealama/thales — [P] [CODE + docs] — H3, H4
- SHA `3c7191328c82`, last commit 2026-08-16, toolchain `leanprover/lean4:v4.33.0`, MIT. Lean files 75, LOC 21,695. Crude counts: sorry 2, axiom 12, native_decide 26, partial 235. No AI-agent files at root.
- README: "emits a Lean 4 sidecar alongside the input `.ts` file". "Every program Thales accepts is also accepted by `tsc --strict`". "Currently out: classes, mutation, async, `any`/`unknown`/intersection types."
- `docs/axioms.md`: "`Thales.TS.Runtime` postulates twelve `axiom`s. They all encode properties of IEEE-754 doubles". README: emitted code reasoning about safe-integer arithmetic "ultimately rests on these."

### E29 PastaLean (Python to Lean) — https://arxiv.org/pdf/2610.34205 — [P] [FULL] — H3, H4
- Deterministic transpiler producing a "provable twin" and a "runnable twin". Table 1 of the paper: HumanEval+ 164 problems, 151 elaborated (92.1%), 7,007 tests all passed; LeetCode 2,589 problems, 2,391 elaborated (92.4%), 215,126 tests, 208,736 passed (97.0%); LiveCodeBench 476 problems, 469 elaborated (98.5%), 10,213 tests all passed. Footnote: the thirteen remaining HumanEval programs elaborate "with one type hint added". The paper says of the two twins (Appendix): "We do not prove that the runnable twin refines the provable one. The two are tied empirically instead, by running the runnable twin against CPython on the dataset's own tests ... and making that tie a proof is future work." Unsupported Python degrades to a `pyUnsupported` placeholder no-op (Appendix A.13); the paper lists `exec`, monkey-patching, `super()` and metaclasses among untranslated features (Appendix A.11 has the full list). Preprint dated 25 Oct 2026 per its arXiv stamp.

### E30 leanlift — https://github.com/zeta1999/leanlift — [P] [CODE + SPEC] — H3, H4, D
- SHA `5a5d6f6ca654`, last commit 2026-10-02, toolchain `leanprover/lean4:v4.28.0`, no licence file. Lean files 79, LOC 11,446. Crude counts: sorry 21, admit 5, axiom 2. No AGENTS/CLAUDE files; README itself is WIP-labelled and describes LLM-generated candidates.
- README: "The trust model is 'LLM proposes, algorithm disposes.' A candidate Lean translation is never trusted". Levels: L0 typechecks, L1 conformant (bit-exact against source oracle on a vector set), L3 proved. Front-ends: LLM for C++/Go/Solidity ("oracle-checked"); cpp2rust + Charon + Aeneas for C++ ("oracle-checked, deterministic"). "float kernels are L1 (testing) only".

### E31 code2lean — https://github.com/phunterlau/code2lean — [P] [CODE] — H3
- SHA `36ae2625495e`, last commit 2026-05-07, toolchain `leanprover/lean4:stable`, MIT. Lean files 93, LOC 4,296. Pipeline: LLM proposer then five gates (sanitizer, Lean compile, axiom allowlist, differential test, second-LLM critic). "Lean is the verifier. The LLM is just a proposer."

### E32 go-in-lean — https://github.com/s-celles/go-in-lean — [P] [CODE + docs] — H3, H4, D
- SHA `e5143a7c0f0b`, last commit 2026-11-07, toolchain `leanprover/lean4:v4.35.0`, MIT. Lean files 69, LOC 19,491. `AGENTS.md` present (AI-agent-assisted indicated). Crude counts: sorry 1, native_decide 1.
- README: "Finding correctness bugs in Go's standard library with Lean 4: a proof-of-concept." Models "a bounded subset of Go (fixed-width integers and, so far, the whole of `math/bits`)", proves documented behaviour with the kernel and `bv_decide`, and turns each failed proof into a Go test. "A failed proof is never reported as a Go bug: every model counterexample must reproduce in Go".
- Reported outcome (web-read): Go issue golang/go#82763, opened 2 Nov 2026 by the repo's author, titled "cmd/compile: wrong result for `((x >> c) & d) << e` with constant shift count `c >= 1<<63`", state open. The repo's own `docs/prior-art.md` contrasts itself with Gobra/Viper and Goose/Perennial.
- The Lean model is hand-written, not extracted from Go source.

### E33 julia-in-lean — https://github.com/MilesCranmer/julia-in-lean — [P] [CODE] — H3, H4, D
- SHA `8c53f618d8bf`, last commit 2026-10-29, toolchain `leanprover/lean4:v4.34.1`, Apache-2.0. Lean files 1,717, LOC 751,381. Crude counts: sorry 0, axiom 1, extern 6, opaque 50.
- README: "Julia source code modelled in Lean 4. Each Julia function is mirrored by a Lean definition that follows the source line by line." Julia Base package: "6,452 registered properties, 5,058 proven, 560 refuted and 834 open (`todo`), with 627 bug write-ups and 107 issues filed on JuliaLang/julia."
- Limits section, verbatim: the model "mirrors the Julia source by hand and is checked against the real implementation by differential testing, not by a verified compiler or a verified semantics of Julia." Coverage partial: "12,774 methods in the manifest, 7,258 modelled, 262 excluded, and 5,254 todo."

### E34 Perennial / Goose (Go to Lean 4) — https://github.com/mit-pdos/perennial (branch `lean`) — [P] [CODE + docs] — H1, H2, H3, H4, D
- Branch `lean`, SHA `84d5380639a3`, last commit 2026-11-05, toolchain `leanprover/lean4:v4.34.1`; no SPDX detected at the root; `goose/` carries its own LICENSE. Lean files 1,679, LOC 348,869. Crude counts: sorry 177 (162 under `Perennial/GeneratedProof`), axiom 3,726 (3,719 under `Perennial/Code`, generated declarations such as `axiom lowerhex [GooseExts] [GoGlobalContext] : val`), implemented_by 1, opaque 30, partial 93. `Perennial/TrustedCode` has about 3,425 Lean LOC. `AGENTS.md`, `CLAUDE.md`, `.claude`, `.agents` present (AI-agent-assisted indicated). Go translator in `goose/` (226 Go files, about 26,000 LOC).
- README: "Perennial's goose framework and program proofs, in Lean 4 on top of iris-lean. This branch (`lean`) is the primary branch. It covers Go programs translated by goose, with a crash semantics and crash adequacy theorem but no crash program logic."
- Distributed systems (README): "A distributed system (`GooseLang/DistLang.lean`) is a list of nodes, each with its own thread pool and local state, sharing the global state". "There is no distributed program logic." End-to-end example: `netfs`, "three nodes on a toy network", with theorem `netfs_crash_safety` whose network primitives "are stubs: the extension is a specification of a network".
- `docs/KNOWN_LIMITATIONS.md` opens: "Places where Perennial's model of Go is known to disagree with Go, or to fall short of it, and that are not fixed yet. A verified program is only verified for the executions the model has, so an entry marked unsound means a program with that behaviour can verify while Go does something else". The file contains 14 entries mentioning "unsound" in text, e.g. a map-key check accepted where Go panics.
- README on trusted code: `Perennial/TrustedCode` supplies initializers for trusted package variables; "five such models are trusted although goose translates the real source".
- Relevance: a Go source → Lean 4 translator with a stated trusted semantics. Whether it is covered by another dossier was not confirmed.

### E35 lean-surfaces — https://github.com/thomasnormal/lean-surfaces — [P] [CODE + README] — H3, H4
- SHA `a077a9ef9e8b`, last commit 2026-11-07, toolchain `leanprover/lean4:v4.33.0-rc1`, Apache-2.0. Lean files 1,192, LOC 327,106. `AGENTS.md` present (AI-agent-assisted indicated). Crude counts: sorry 40 (admit count of 682 is vocabulary matches).
- README: "Prove real Python / C++ / Rust / SystemVerilog programs correct in Lean 4. Each language gets a deep embedding: the program's real AST becomes a Lean value, and a definitional interpreter gives it meaning." Table of per-language "differential rows": Python 4,174 / 4,188; C++ 1,565 / 1,582; SystemVerilog 616 / 639; Rust "Lane planned, implementation not started".
- "Unvalidated lanes. The Go, ES (JavaScript), Ada, SPICE and Verilog-A lanes are unvalidated: an audit ... on 2026-11-05 found silent wrong answers in each (issue #11). Do not rely on their results until that issue is closed."

### E36 LeanPort — https://github.com/shrsv/LeanPort — [P] [CODE] — H3
- SHA `a13b2245186e`, last commit 2026-10-25, toolchain `leanprover/lean4:4.29.1`, MIT. Lean files 22, LOC 479. Direction is Lean → Go: "turns a folder of Lean 4 modules into a cgo-consumable Go package ... checks the proofs, compiles the Lean implementation to C, packages a native static archive, and generates the C/cgo bridge plus an idiomatic Go API."

### E37 lean-zip — https://github.com/kim-em/lean-zip — [P] [CODE + README] — H3, D
- SHA `2f7a63f38195`, last commit 2026-09-23, toolchain `leanprover/lean4:v4.35.0-rc2`, Apache-2.0. Lean files 114, LOC 73,935. Crude counts: sorry 1, implemented_by 15, extern 61, opaque 29. `.claude` directory present.
- README: "A formally verified DEFLATE implementation in Lean 4 ... contains a pure-Lean DEFLATE encoder and decoder, and Lean's kernel checks that they are inverse to each other." Verbatim theorem: `theorem zlib_decompressSingle_compress (data : ByteArray) (level : UInt8) (maxOutputSize : Nat) (hsize : data.size ≤ maxOutputSize) : ZlibDecode.decompressSingle (ZlibEncode.compress data level) maxOutputSize = .ok data`. "more than 1,100 theorems across ~32k lines of proof". "both the implementation, and the verification, are written entirely by loosely supervised AIs" (README self-statement).
- The `extern` count reflects native bindings in the repo (including zlib-related FFI code); the README's theorem concerns the pure-Lean implementation.

---

## 5. CompCert-style and verified-compilation routes

### E38 CompCert motivations and manual — https://compcert.org/motivations.html ; https://compcert.org/man/manual001.html — [P] [FULL] — H3, D
- CompCert is written and proved in Coq/Rocq. Manual page: "the executable code it produces is proved to behave exactly as specified by the semantics of the source C program". Both pages state: "complete, end-to-end formal verification has not been achieved yet" in the sense that "the remaining 10% (including elaboration, presimplifications, assembling and linking) are not verified" (about 90% of the compiler's algorithms are proved in Coq/Rocq). Not a Lean artifact.

### E39 "The Trusted Computing Base of the CompCert Verified Compiler" — https://ar5iv.labs.arxiv.org/html/2201.10280 — [P] [FULL, abstract and introduction] — D, H3
- Dedicated TCB analysis of CompCert. Introduction states the headline guarantee: "if the compiler succeeded in compiling a C program and that program executes with no undefined behavior, then the assembly code produced executes correctly with the same observable behavior. Yet, this impressive claim comes with some caveats". Included as a reference point for "verified compilation" TCB statements; not a Lean artifact.

### E40 lean-compcert — https://github.com/gersh/leancompcert — [P] [CODE + docs] — H3, H4, D
- SHA `dfc7ae2171e8`, last commit 2026-08-31, toolchain `leanprover/lean4:v4.32.1`, no root licence file. Lean files 6,129, LOC 1,628,183 (includes much generated/vendored content). Crude counts: sorry 10, axiom 62, native_decide 74. Repo includes Coq/CompCert scripts.
- README: "lean-compcert turns finite Lean computations into kernel-checked theorems and CompCert-compiled native artifacts." Docs `what-is-proved.md`: a "program" here "is not arbitrary Lean code and not arbitrary C. It is a value of a small first-order datatype — a bounded fold over a register machine".
- README "Scope and honest self-description": "This project describes itself as a Lean native backend and verified-computation package that obtains CompCert's verified C-to-assembly guarantee, with kernel-checked certificates and a Coq-kernel-checked Clight semantics theorem for its certificate fragment — not as a formally verified compiler of arbitrary Lean programs to machine code." Trusted: "Lean's elaborator and kernel, Coq's kernel, and CompCert's stated assumptions (assembler, linker, hardware)".

### E41 turbo-lean — https://github.com/Th0rgal/turbo-lean — [P] [CODE + README] — H3, D
- SHA `aa71c7ed4db2`, last commit 2026-11-07, toolchain `leanprover/lean4:v4.35.0`, MIT. Lean files 219, LOC 112,258. `AGENTS.md` present (AI-agent-assisted indicated). Crude counts: sorry 11, admit 2, native_decide 15, implemented_by 23, extern 41.
- README claims "a formally verified native compiler (`LeanNative`)" lowering `Lean.Compiler.LCNF` through `LNIR` to native code; semantic refinement is stated "against the in-repo `RealLCNFSemantics` evaluator ... for the supported scalar, flat-tuple, and bounded Stage-1D constructor/ADT subset". Wiring to upstream tools remains roadmap. Scope: a supported subset; not a proof about Lean's shipped compiler.

---

## 6. Repository statistics table (CODE entries)

All data from shallow clones taken 2026-11-07; counts are crude greps (section 0). "AI files" = `AGENTS.md`/`CLAUDE.md`/`.claude`/`.agents` observed.

| Repo | SHA | Last commit | Toolchain | Lean LOC | sorry | axiom | native_decide | implemented_by | extern | opaque | partial | AI files |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| LeanCIL | f96b016a7c6b | 2026-11-05 | v4.35.0 | 80,084 | 5 | 1 | 0 | 3 | 0 | 72 | 29 | AGENTS |
| Clear | 6ef2e7eda010 | 2025-08-07 | v4.9.1 | 6,227 | 0 | 0 | 0 | 0 | 0 | 0 | 8 | no |
| EVMYulLean | 047f63070309 | 2025-09-24 | v4.22.0 | 11,339 | 1 | 0 | 0 | 1 | 4 | 4 | 16 | no |
| Verity | 16e84d8d1182 | 2026-11-07 | v4.31.0 | 362,938 | 29 | 12 | 579 | 24 | 12 | 44 | 404 | AGENTS, CLAUDE |
| LemmaScript | 385cfbdf4ffb | 2026-11-04 | v4.34.0 | 3,755 | 0 | 0 | 0 | 0 | 0 | 1 | 0 | AGENTS |
| Thales | 3c7191328c82 | 2026-08-16 | v4.33.0 | 21,695 | 2 | 12 | 26 | 0 | 0 | 2 | 235 | no |
| leanlift | 5a5d6f6ca654 | 2026-10-02 | v4.28.0 | 11,446 | 21 | 2 | 0 | 0 | 1 | 2 | 10 | no |
| go-in-lean | e5143a7c0f0b | 2026-11-07 | v4.35.0 | 19,491 | 1 | 0 | 1 | 0 | 0 | 0 | 0 | AGENTS |
| code2lean | 36ae2625495e | 2026-05-07 | stable | 4,296 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | no |
| julia-in-lean | 8c53f618d8bf | 2026-10-29 | v4.34.1 | 751,381 | 0 | 1 | 0 | 0 | 6 | 50 | 75 | not checked |
| Perennial (lean) | 84d5380639a3 | 2026-11-05 | v4.34.1 | 348,869 | 177 | 3,726 | 0 | 1 | 0 | 30 | 93 | AGENTS, CLAUDE, .claude, .agents |
| lean-surfaces | a077a9ef9e8b | 2026-11-07 | v4.33.0-rc1 | 327,106 | 40 | 0 | 30 | 1 | 17 | 43 | 343 | AGENTS |
| ship-your-interpreter | fb07094cff91 | 2026-10-01 | v4.34.0 | 421,771 | 1 | 75 | 0 | 0 | 0 | 1 | 93 | no |
| lean-compcert | dfc7ae2171e8 | 2026-08-31 | v4.32.1 | 1,628,183 | 10 | 62 | 74 | 0 | 2 | 74 | 201 | no |
| turbo-lean | aa71c7ed4db2 | 2026-11-07 | v4.35.0 | 112,258 | 11 | 0 | 15 | 23 | 41 | 50 | 124 | AGENTS |
| LeanPort | a13b2245186e | 2026-10-25 | 4.29.1 | 479 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | no |
| lean-zip | 2f7a63f38195 | 2026-09-23 | v4.35.0-rc2 | 73,935 | 1 | 0 | 0 | 15 | 61 | 29 | 29 | .claude |
| Lean4Lean | 67f63b0c8189 | 2026-10-10 | v4.33.0-rc2 | 49,412 | 131 | 73 | 0 | 14 | 0 | 36 | 48 | no |

---

## 7. Feature coverage, as documented (no inference)

Legend: Y documented supported; N documented unsupported; ? not stated in what was read.

| Item | Aeneas/Lean (E14,E13) | hax→Lean (E17) | LeanCIL (E22) | Thales (E28) | PastaLean (E29) | Goose/Perennial (E34) | Std.Do/vcgen (E06,E05) |
|---|---|---|---|---|---|---|---|
| Loops | Y (nested return/break limited) | Y via Aeneas (E18: recursion + measure) | ? | ? | Y | ? | Y (invariant required) |
| Mutation | Y (functional model) | Y | Y (checked memory) | N | ? | Y | Y (state monads) |
| Unsafe/raw pointers | N | N | ? | n/a | n/a | model of Go memory | n/a |
| Concurrency | N | N | N | N (async) | N | Y (thread pool semantics, no distributed logic) | not addressed |
| Exceptions/panics | Y | Y | N (handlers) | Y (`@throws`) | Y | Y | Y (`ExceptT`) |
| Floating point | ? | ? | N | Y with 12 axioms | ? | ? | Lean `Float` |
| Dynamic dispatch | N | N | N | N | N | ? | n/a |
| Network/distributed | N | N | N | N | N | toy network only | not addressed |

---

## 8. Conflicts (unresolved)

1. Compiler trust: E01 states native-evaluation proofs depend on "the correctness of the entire compiler"; E12 (SampCert) states the deployed artifact does not go through the Lean compiler. Both are statements about different routes; neither source relates them.
2. trustCompiler: Phase 1 recorded a 2023 community report that `Lean.reduceBool` leakage was fixed by an axiom (E10); the current reference (E02) says the `Lean.trustCompiler` machinery was removed in 4.35.0. The replacement mechanism per E01 is a bespoke axiom per `native_decide` call. The sources do not reconcile the history.
3. Aeneas and Verity both state "zero axioms/sorry" for proof stacks, while crude greps over the same repos find matches (Verity: axiom 12, sorry 29; see section 0 on method). Verity's docs scope the claim to the compiler proof stack. Whether the matches fall outside that scope was not individually audited.
4. Velvet/Loom venue and status: one source lists Loom at POPL 2026 and Velvet as a CAV 2026 prototype; the Loom ACM page returned an access wall in Phase 1b (lead L12). Not resolved here; Velvet/Loom dossier owns it.
5. Verification terminology: lean-zip, Perennial and Aeneas use "verified" for theorem-level kernel-checked statements; Cedar (per E12), julia-in-lean (E33), PastaLean (E29) and leanlift (E30) use differential testing to connect a Lean model to code. julia-in-lean states that a proof "says: if the model is faithful on the inputs the property quantifies over, the property holds." The sources do not agree on a common name for the two situations.
6. lean-zip: Phase 1 secondary summary said an FFI zlib was trusted; the README read in Phase 1b describes a pure-Lean implementation (E37).
7. hax Lean maturity: E17 says "legacy-lean" remains while the new backend delegates to Charon and Aeneas; E18 (a paper on the older state) says the Lean backend is "under active development" and cites semantic gaps. Dating of E18 relative to hax 0.4 is not stated.

---

## 9. Quote verification

Method: an automated pass extracted every quoted string of 25 or more characters (141 strings) and tested for an exact, whitespace-normalised match against the downloaded page texts and repository files. Mismatches found in that pass were corrected in the text (examples: Lean4Lean, Aeneas ICFP sentence, PastaLean Table 1 figures and appendix quote, Scylla wording, trustCompiler paraphrase). 23 strings remained unmatched by the script; they were checked by hand and fall into these groups: (a) quotes containing an ellipsis that joins two non-adjacent passages of the same source; (b) strings containing markdown link syntax or inline code spans that the plain-text source lacks; (c) text read on a rendered web page (for example the GitHub archived-repository banner, the Lean release notes, README sections) that is not in the downloaded text files; (d) fragments cut by the script at a quote boundary. This hand check was not a second exact-match run, so it is weaker than the automated pass. Quotes in group (c) are the least mechanically verified.

---

## 10. Phase 1 gaps: status

| Phase 1 gap | Status after Phase 1b |
|---|---|
| No sound C→Lean extractor | Remains. Found: Scylla (C→Rust, E15), C binary → Sail RISC-V model (E24, one interpreter), lean-surfaces C++ deep embedding (E35), leanlift LLM route (E30). None is a general C extractor. |
| No Go→Lean extractor | Closed in part: Perennial/Goose has a Lean 4 branch with a Go translator and a trusted semantics (E34); go-in-lean (E32) is hand-modelled. |
| Move verification in Lean | Remains. Only the borrow-checker slide (E12) found. |
| No Aeneas/hax extraction-success rates | Remains. |
| No hax TCB statement | Remains for the Lean backend (E17). |
| LNSym instruction subset | Remains (E20 README has no list). |
| Lean compiler formal verification | Remains: none found. turbo-lean (E41) claims a verified compiler for a subset; its claims relate to an in-repo evaluator. |

---

## Open Questions

1. Aeneas: which parts of the Rust→Lean path (Charon extraction, LLBC semantics, Lean emission) have proofs, and which are trusted (E14, E16)?
2. hax Lean backend 0.4: any published TCB or validation beyond E17(c)?
3. Corpus-level extraction success rate for Charon/Aeneas on real crates (none in E13, E18).
4. Goose/Perennial (E34): which Go language and library features are outside the translator? How large are programs proved so far? The KNOWN_LIMITATIONS file lists model-vs-Go disagreements; its completeness is unknown.
5. Does any source connect a Lean protocol specification to Go/Rust/C implementations of a network protocol end to end? E34's `netfs` example uses a toy network with stubbed primitives.
6. Differential testing as the link (Cedar via E12, E33): any published bug-finding statistics?
7. LemmaScript (E27): how much of the Lean backend's coverage differs from the Dafny backend, and is the "inspection" validation of the code generator ever replaced by proof?
8. Verity (E26): are the 12 `axiom` and 29 `sorry` matches outside the compiler proof stack?
9. Lean compiler: are there plans or results for verifying LCNF/IR/C emission (E05, E11)? turbo-lean (E41) states a goal.
10. SampCert / Cedar / Velvet / Loom / LNSym details are in separate dossiers; consistency of terminology between them and this file has not been checked.

---

## Unverified leads (excluded from evidence)

Not read in full, or not re-read in Phase 1b. Phase 2 should ignore.

- L01 Trivet (LLVM translation validation with Lean-MLIR) https://ar5iv.labs.arxiv.org/html/2609.19583 — downloaded, only abstract examined.
- L02 Counting Immutable Beans (Lean IR design) https://arxiv.org/html/1908.05647v3 — not read.
- L03 Lean 4 system description (Springer) https://link.springer.com/chapter/10.1007/978-3-030-79876-5_37 — snippet only.
- L04 CompCert TCB blog summary https://www.cs.cornell.edu/courses/cs6120/2023fa/blog/compcert/ — not read.
- L05 Nethermind formal verification page (EVMYulLean test pass rate 99.99%) https://www.nethermind.io/formal-verification — snippet only.
- L06 Nethermind Clear blog https://www.nethermind.io/blog/clear-prove-anything-about-your-solidity-smart-contracts — snippet only.
- L07 Verity paper/refinement proofs https://ar5iv.labs.arxiv.org/html/2607.26306 — snippet only.
- L08 Verity FAQ and verification pages https://veritylang.com/faq — fetched, not used.
- L09 Shielded-pool writeup https://ethsystems.org/writeups/formally-verifying-a-shielded-pool-from-spec-to-bytecode/ — snippet only.
- L10 FVSpec https://arxiv.org/pdf/2606.01008v1 — snippet only.
- L11 Vero https://ar5iv.labs.arxiv.org/html/2608.13522 — snippet only.
- L12 Loom ACM page https://dl.acm.org/doi/10.1145/3776719 — access wall; Phase 1 text came from an LLM-extracted summary.
- L13 Velvet repo and site https://github.com/verse-lab/velvet — in the Velvet dossier.
- L14 Cedar security page and AWS blog https://docs.cedarpolicy.com/other/security.html ; https://aws.amazon.com/blogs/opensource/lean-into-verified-software-development/ — downloaded, not read in Phase 1b; Cedar dossier owns.
- L15 SampCert paper https://arxiv.org/html/2412.01671 — SampCert dossier owns.
- L16 jxl-rs case study https://jonathan.protzenko.fr/2026/05/05/jxl-rs.html — downloaded, not read in Phase 1b.
- L17 Move Prover paper https://ar5iv.labs.arxiv.org/html/2110.08362 — Boogie/SMT, not Lean; snippet only.
- L18 StanfordLeanClub/c-py-dataset https://huggingface.co/datasets/StanfordLeanClub/c-py-dataset — snippet only.
- L19 Skills-4-SE "python-to-lean4-translator" prompt https://github.com/ArabelaTso/Skills-4-SE — LLM prompt text, not a tool; snippet only. Also zksecurity/leanPython plan, rust-core-models, Aeneas ICFP paper (abstract only), Aeneas newsletter, `lean2agda`, Lean Kernel Arena.
