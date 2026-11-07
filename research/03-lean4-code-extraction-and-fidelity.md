# 03 – Lean 4 code extraction and fidelity

Phase 1 evidence file. No analysis or conclusions. Facts are recorded as the sources state them.

- Access date for every URL below: **2026-11-07** (content as served on that date; many pages are living documents).
- Tags: **[P]** = primary (tool's own repository, documentation, paper by the tool's authors, or official Lean documentation). **[S]** = secondary (third-party description, blog, aggregator, community post).
- "Snippet only" = the fact comes from a search-result excerpt and the full page was not read.
- Quotes are verbatim from the fetched or excerpted text.
- Scope of this file: how Lean 4 connects to real implementation code. Protocol-specific modelling is covered in files 01, 02, 07, 08; where a source mentions protocols in passing, it is recorded in section 12.

---

## 0. Source index

| ID | URL | Tag |
|---|---|---|
| S1 | https://github.com/AeneasVerif/aeneas | P |
| S2 | https://aeneasverif.github.io/projects/ | P |
| S3 | https://arxiv.org/abs/2206.07185 (Aeneas, ICFP 2022) | P |
| S4 | https://arxiv.org/html/2410.18042v2 (Charon) | P |
| S5 | https://arxiv.org/html/2609.15648v1 (SymCrypt with Aeneas, Lean) | P |
| S6 | https://jonathan.protzenko.fr/2026/05/05/jxl-rs.html (jxl-rs case study) | P (snippet only) |
| S7 | https://hax.cryspen.com/manual/ | P |
| S8 | https://hax.cryspen.com/manual/lean/quick_start/ | P (snippet only) |
| S9 | https://hax.cryspen.com/blog/2026/09/29/hax-04-easier-to-install-and-configure-and-a-new-lean-backend/ | P |
| S10 | https://hax.cryspen.com/dev/architecture/ and https://cryspen.com/post/hax_0_4_release/ | P (snippet only) |
| S11 | https://arxiv.org/html/2605.30106 (Rust-to-Lean pipeline with AI provers) | S (third-party evaluation) |
| S12 | https://github.com/cryspen/rust-core-models | P (snippet only) |
| S13 | https://github.com/AeneasVerif/eurydice | P (snippet only) |
| S14 | https://github.com/leanprover/LNSym | P |
| S15 | https://www.amazon.science/blog/how-the-lean-language-brings-math-to-coding-and-coding-to-math | P (Lean author, Amazon) |
| S16 | https://lean-lang.org/doc/reference/latest/Run-Time-Code/Foreign-Function-Interface/ | P |
| S17 | https://lean-lang.org/doc/reference/latest/Axioms/ | P |
| S18 | https://arxiv.org/html/2403.14064 (Lean4Lean) | P |
| S19 | https://ammkrn.github.io/type_checking_in_lean4/trust/trust.html | S |
| S20 | https://leanprover-community.github.io/archive/stream/270676-lean4/topic/soundness.20bug.3A.20native_decide.20leakage.html | S (community thread) |
| S21 | https://lean-lang.org/doc/reference/latest///Run-Time-Code/Reference-Counting/ and https://lean-lang.org/doc/api/Lean/Compiler/IR/Basic.html | P |
| S22 | https://lean-lang.org/doc/reference/4.36.0-rc1/releases/v4.30.0/ | P |
| S23 | https://arxiv.org/html/1908.05647v3 (Counting Immutable Beans) | P |
| S24 | https://link.springer.com/chapter/10.1007/978-3-030-79876-5_37 (Lean 4 system description) | P (snippet only) |
| S25 | https://lean-lang.org/doc/reference/latest/The--mvcgen--tactic/ , .../Overview/ , https://lean-lang.org/doc/tutorials/latest/mvcgen/ | P |
| S26 | https://lean-lang.org/doc/reference/latest/releases/v4.22.0/ ; https://lean-lang.org/doc/reference/stable/releases/v4.35.0/ ; https://lean-lang.org/doc/reference/latest/releases/v4.31.0/ | P |
| S27 | https://dl.acm.org/doi/10.1145/3776719 (Loom, "Foundational Multi-Modal Program Verifiers") | P |
| S28 | https://github.com/verse-lab/velvet ; https://velvetprover.dev/ | P |
| S29 | https://leodemoura.github.io/static/nfm2026/ (Lean 4: A Unified Platform, talk) | P (Lean author slides) |
| S30 | https://docs.cedarpolicy.com/other/security.html ; https://arxiv.org/html/2407.01688v1 ; https://github.com/cedar-policy/cedar-spec | P |
| S31 | https://lean-lang.org/use-cases/cedar/ | P |
| S32 | https://arxiv.org/html/2412.01671 (SampCert) | P (snippet only) |
| S33 | https://www.nethermind.io/blog/clear-prove-anything-about-your-solidity-smart-contracts ; https://github.com/NethermindEth/Clear | P |
| S34 | https://ethsystems.org/writeups/formally-verifying-a-shielded-pool-from-spec-to-bytecode/ | S |
| S35 | https://veritylang.com/faq ; https://veritylang.com/verification ; https://github.com/lfglabs-dev/verity/blob/main/README.md | P |
| S36 | https://www.nethermind.io/formal-verification | P |
| S37 | https://github.com/NethermindEth/LeanCIL/blob/main/README.md | P |
| S38 | https://github.com/midspiral/LemmaScript | P |
| S39 | https://reservoir.lean-lang.org/@jessealama/thales/ | P |
| S40 | https://arxiv.org/pdf/2610.34205 (PastaLean) | P |
| S41 | https://github.com/zeta1999/leanlift/blob/leanlift-engine/SPEC.md | P (draft spec) |
| S42 | https://github.com/phunterlau/code2lean | P (snippet only) |
| S43 | https://github.com/thomasnormal/lean-surfaces/blob/master/README.md | P (snippet only) |
| S44 | https://github.com/s-celles/go-in-lean | P (snippet only) |
| S45 | https://github.com/shrsv/LeanPort | P (snippet only) |
| S46 | https://github.com/Beneficial-AI-Foundation/probe-lean/blob/main/docs/lean-verification-landscape.md | S |
| S47 | https://compcert.org/motivations.html ; https://compcert.org/man/manual001.html | P |
| S48 | https://ar5iv.labs.arxiv.org/html/2201.10280 (TCB of CompCert) ; https://www.cs.cornell.edu/courses/cs6120/2023fa/blog/compcert/ | P / S |
| S49 | https://github.com/gersh/leancompcert | P (snippet only; experimental) |
| S50 | https://ar5iv.labs.arxiv.org/html/2609.19583 (Trivet, LLVM translation validation with Lean-MLIR) ; https://llvm.org/devmtg/2024-10/ | P (snippet only) |
| S51 | https://github.com/Th0rgal/turbo-lean/blob/main/README.md | P (snippet only; repo claims) |
| S52 | https://arxiv.org/pdf/2606.01008v1 (FVSpec) | P (snippet only) |
| S53 | https://ar5iv.labs.arxiv.org/html/2110.08362 (Move Prover) | P (snippet only) |
| S54 | https://github.com/zksecurity/leanPython/blob/main/PLAN.md | P (snippet only; plan) |
| S55 | https://github.com/ArabelaTso/Skills-4-SE/blob/main/skills/python-to-lean4-translator/SKILL.md | P (LLM prompt "skill", not a tool) |
| S56 | https://aws.amazon.com/blogs/opensource/lean-into-verified-software-development/ | P |
| S57 | https://www.microsoft.com/en-us/research/blog/rewriting-symcrypt-in-rust-to-modernize-microsofts-cryptographic-library/ | P (snippet only) |
| S58 | https://leanprover-community.github.io/archive/stream/270676-lean4/topic/The.20state.20of.20the.20code.20generator.html | S (community thread) |

---

## 1. Rust → Lean: Charon + Aeneas

### 1.1 What the tools are

- Aeneas README [S1, P]: "Aeneas (pronunced [Eh-nee-as]) is a verification toolchain for Rust programs. It relies on a translation from Rust's MIR internal language to a pure lambda calculus. It is intended to be used in combination with Charon, which compiles Rust programs to an intermediate representation called LLBC. It currently has backends for F*, Coq, HOL4 and LEAN."
- Charon [S2, P]: "implemented as a custom driver for the rustc compiler". Pipeline: MIR → ULLBC (Unstructured Low-Level Borrow Calculus) → control-flow reconstruction → LLBC (Low-Level Borrow Calculus). JSON serializers exist for both, plus an OCaml library `charon-ml`.
- Aeneas [S2, P]: translates LLBC into "a pure, functional model"; "currently targets a *safe*, sequential Rust code."
- Aeneas Lean backend [S1, S2, P]: "Our most mature backends are Lean and HOL4, for which we have in particular support for partial functions and extrinsic proofs of termination … and tactics specialized for monadic programs".
- Aeneas generated Lean code [S46, S]: "Aeneas translates safe Rust into pure functional Lean code in the `Result α` monad (constructors: `ok`, `fail`, `div`)." "The generated code contains no specs — it is purely functional." "Specs are written as separate theorems by the proof engineer." Aeneas's `progress` tactic looks up registered (`@[progress]`) specs by pattern-matching on the function call.
- Workflow [S6, P, snippet only]: Charon is run with `--start-from` to extract a target function and its transitive dependencies, then Aeneas emits Lean; "Aeneas is tied to a particular version of Lean: we adopt the lean-toolchain provided by the Aeneas repository."

### 1.2 Theory / formal status

- ICFP 2022 paper [S3, P]: "We leverage Rust's rich region-based type system to eliminate memory reasoning for many Rust programs, as long as they do not rely on interior mutability or unsafe code." "Our semantics is value-based, meaning there is no notion of memory, addresses or pointer arithmetic." The abstract page does not state a translation-correctness theorem.
- Aeneas README [S1, P]: "The translation has been formalized and published at ICFP2022" and "We also have a proof that the symbolic execution performed by Aeneas during its translation correctly implements a borrow checker, and published it at ICFP 2024". The README states no trusted-assumption list.
- Charon [S4, P]: presented as "an analysis framework"; the paper "does not claim that Charon itself is formally verified" (this is a reading of the paper by the fetch, not a verbatim quote).

### 1.3 Documented limitations

From the Aeneas README [S1, P]:
- "Aeneas currently functionalizes a subset of safe Rust, and we are in the process of extending the functional model with reasoning based on separation logic to support unsafe code and concurrent code."
- Loops: "`return` inside *nested* loops, or `break`/`continue` to *outer* loops are not supported yet … This is a technical limitation, not a fundamental issue".
- Mutable references and generics: "Instantiating a generic type (including `Self` through a trait implementation) with a mutable reference is not supported yet".
- "The following limitations will be lifted by the ongoing work on separation logic: unsafe code; concurrency".

From the Charon paper [S4, P]:
- "some less-central features such as generic associated types, dyn Trait, or trait aliases are not yet fully supported."
- "a declaration that cannot be translated is marked as missing and the rest of the crate is translated as usual, resulting in a well-formed (if incomplete) (U)LLBC."
- "Aeneas comes with restrictions, notably, the programmer cannot rely on runtime-checked aliasing (RefCell) or concurrency."
- Depends on unstable rustc internals; requires a nightly toolchain (paper statement as summarised by fetch).
- Evaluation numbers: constant-time taint analysis on RustCrypto MACs (1,700 lines), a safe-Rust HACL* port (~84 kLoC), libcrux ML-KEM (~12 kLoC); Charon→LLBC averaged 80 s. No single total of crates translated is given.

From the SymCrypt paper [S5, P]:
- "Aeneas has limited support for unsafe Rust and does not support raw pointers, interior mutability, or concurrency; functions using unsupported features must be modelled by hand."
- Unsupported functions are marked `verify::opaque` (Aeneas then emits only a type signature) and given a hand-written Lean model or axiom; example: `wipe_slice` (raw pointers and external code) is given an axiom that sets every slice element to zero.

From a third-party evaluation [S11, S]:
- "The supported subset of safe Rust that Aeneas and Hax can extract to pure Lean is finite."
- "Generic functions with trait bounds … cannot be extracted directly: Aeneas needs concrete monomorphised types."
- "External-crate calls (e.g., byteorder, std::io::Read/Write) are not extractable because Charon does not see the MIR of crates beyond the workspace."
- "Rust while loops become recursive functions in Lean and require explicit termination measures."
- "Casts to wider integer types (u128) inside modular arithmetic require careful proof engineering".
- No extraction success/failure rate is given in that paper.

Charon/Aeneas error handling [S, newsletters, snippet only]: Aeneas newsletter May 2025 (https://aeneasverif.github.io/newsletter/2025/05/15/aeneas-newsletter.html): "make Aeneas more resilient to errors happening either when extracting the crate with Charon or during the translation (because of unsupported features in particular)".

### 1.4 Largest documented use with Lean: SymCrypt (Microsoft)

[S5, P] (Ho, Fournet, Protzenko, Naehrig):
- Verified: Rust code in SymCrypt: SHA-3, ML-KEM; experimental FrodoKEM, ML-DSA, HPKE, HMAC/HKDF, SHA-2. "237 KLOC" of Lean establishing "safety, panic-freedom, and functional correctness" for "16.7 KLOC of Rust code", targets x86-64 and AArch64.
- Not covered: "We verify only functional correctness, not leakage resistance." C FFI code is outside scope.
- Stated trust list (verbatim): "we trust the Rust toolchain, notably its compiler and its platform-specific back-ends for x86-64 (including SSE2/AVX2) and AArch64 (including NEON)"; trust "that Charon correctly extracts MIR into LLBC"; "that Aeneas correctly translates LLBC into Lean"; "the soundness of the Lean kernel that checks all proofs in our development"; "the soundness of the compiled native_decide and bv_decide tactics"; "the soundness of the Aeneas axiomatization for some functions of the Rust standard library".
- Specifications: "we trust the Lean formal specification faithfully captures the standard". Spec developed "independently of the Rust implementation and its correctness proofs"; tested against official known-answer vectors. Hardware intrinsics are modelled in Rust shims, reviewed against vendor docs and differentially tested against real intrinsics where hardware allows.
- Lean proofs: authors say they "need not trust the Lean proofs" (checked by the kernel).

### 1.5 Related Aeneas-ecosystem tools (not Lean-targeting)

- Eurydice [S13, P, snippet only]: Rust→C compiler. "Eurydice's informal guarantee is: if the Rust program terminates without panicking, then the Eurydice-compiled C code computes the same result without undefined behavior. If the Rust program panics … no guarantees are made about the behavior of the generated C code." SymCrypt plans to check generated C into its repository [S57, P, snippet only].
- Scylla [S2, P]: C→safe-Rust translator, "highly regular C code"; as of July 2025 "could translate parts of the HACL* and SymCrypt cryptographic libraries, the CBOR binary parser from EverParse, and the core compression algorithm of bzip2", requiring "different levels of manual C-source rewritings and annotations". Rust output can then be fed to Charon/Aeneas.

---

## 2. Rust → Lean: hax (Cryspen)

- Intro [S7, P]: "hax is a tool for high assurance translations of a large subset of Rust into formal languages such as Lean, F* or Rocq." The page does not list unsupported features and contains "no explicit trust statements about hax translation correctness" (reading of the fetch).
- Backends listed [S7]: `lean`, `legacy-lean`, `fstar`, `coq`, `pro-verif`, `ssprove`, `easycrypt`.
- hax 0.4 [S9, S10, P]: "The new Lean backend does not use the hax engine. Instead, the `cargo-hax` command line tool drives Charon and Aeneas, which translate Rust into Lean." Release post: "we decided in early 2026 to pivot and directly use the Charon+Aeneas pipeline as the translation engine for our Lean backend" and this "increases support for Rust, notably *functions returning mutable borrows*". The earlier Lean backend remains as `legacy-lean`.
- hax architecture [S10, P, snippet only]: the legacy engine pipeline exports rustc THIR as JSON, then an engine lowers it per backend.
- Specifications [S9, P]: `hax_lib::requires` / `hax_lib::ensures` become Lean pre/postconditions, combined into a Hoare-triple-style specification: "if the precondition holds, the function returns without failing and its result satisfies the postcondition." Functions are translated into a `RustM` monad "which models failures such as panics and integer overflow". For each spec hax generates a theorem whose initial proof is `sorry`; the finished example uses `hax_mvcgen` and `grind`.
- Limitations stated [S8, P, snippet only]: "hax doesn't support all Rust constructs, e.g, `unsafe` code or interior mutability." Quick start suggests extracting only part of a crate (`--charon-args="--start-from …"`).
- Version coupling [S9]: other combinations of Charon/Aeneas/Lean/hax-Lean-library versions "are untested and must be maintained by users".
- Library model [S12, P, snippet only]: `cryspen/rust-core-models` provides Rust crates mirroring `core::*`/`alloc::*`, plus a Lean library extracted by Aeneas; includes a "rust↔lean equivalence" test framework in which "Each test pins one concrete behavioural observation … checked once on the Rust side via `cargo test` and once on the Lean side via a `#guard`".
- Maturity statements: hax 0.4 post [S9]: "Our main goal for the next release is robustness. We want hax to work on almost any crate with all backends." A third-party paper [S11, S] says "The Lean 4 backend is currently under active development, while the F^⋆^ backend is more mature" and "Recent independent analysis [23] has identified semantic gaps between Hax extractions and the original Rust semantics for certain Rust patterns" (reference [23] not identified in the excerpt; this predates the 0.4 Aeneas-based backend per the paper's framing, but the paper does not state this explicitly).
- Trust [S9]: "The post does not specify a formal trust model, trusted computing base, or independent validation of the translation" (fetch reading). hax verifies downloaded Charon/Aeneas binaries against a manifest shipped with the release.

---

## 3. Native-code route: LNSym (Armv8 machine code)

- [S14, P] "LNSym is described as 'a symbolic simulator for Armv8 machine-code programs.'" Repository includes `Arm` ("Formalization of the Armv8 Aarch64 ISA"), specifications of algorithms, proofs of Arm native-code programs, and concrete tests. The README does not list supported instructions or binary-loading details (fetch reading). A `cosim` target performs conformance testing: "if you are not on an Aarch64 machine, conformance testing will be skipped."; default 20 random tests per instruction class. `awslc_elf` target: "ELF loading tests for AWS-LC".
- [S15, P] (Amazon Science, Leo de Moura): "It's currently under development, with a focus on enabling automated reasoning of cryptographic machine-code programs." "LNSym uses Lean as a specification language to model the Arm instruction semantics and cryptographic protocols and as a theorem prover for reasoning about these artifacts. Since Lean programs are executable, the specifications achieve a high degree of trust through thorough conformance testing." Automation: "decision procedures like SAT solvers or custom domain-specific tactics".
- Reservoir listing (https://reservoir.lean-lang.org/@leanprover/LNSym, snippet only) uses the same description.

Other machine/bytecode-level Lean models found:
- LeanCIL [S37, P]: Lean 4 proofs about compiled .NET Common Intermediate Language. Supported: "integer arithmetic, selected control flow and managed calls, reference-free structs, checked memory access, and arrays and spans of integers …". Limitations: "General class allocation and virtual dispatch, ordinary recursive calls, exception handlers, floating-point arithmetic and arbitrary external calls are outside the supported subset." "Concurrency, GC implementation correctness and equivalence to arbitrary .NET runtimes are outside the model's guarantees." "The extractor, CIL model and verification machinery have an explicit trust boundary; LeanCIL does not claim complete CLR or JIT verification." "Unsupported instructions and signatures are rejected explicitly."
- EVMYulLean [S36, P]: Nethermind's executable Lean semantics of the EVM and Yul (Cancun), "passing 99.99% of official Cancun execution tests". A 2026 arXiv paper (https://ar5iv.labs.arxiv.org/html/2607.26306, snippet only, P) builds "Foundational Refinement Proofs for Deployed Bytecode" on it.
- Lean-MLIR / LLVM dialect [S50, P, snippet only]: "a Lean 4 framework that formalizes the semantics of MLIR programs"; Trivet uses it with LLMs for LLVM translation validation.

---

## 4. Lean's own compiler, runtime and trusted base

### 4.1 Compilation path

- Lean 4 system description [S24, P, snippet only]: "The new compiler produces C code, and users can now implement efficient proof automation in Lean, compile it into efficient C code, and load it as a plugin."
- Pipeline described in the Lean 4 repository documentation (DeepWiki summary, https://deepwiki.com/leanprover/lean4/6-compiler-backend, S, snippet only): elaborated `Expr` → LCNF → optimisation passes → IR → C (or LLVM IR).
- Lean API docs [S21, P]: "The Lean to IR transformation produces λPure code, and this part is implemented in C++. The procedures described in the paper above are implemented in Lean."
- Reference manual [S21, P]: Lean uses reference counting; IR makes "reference counting, allocation, and reuse … explicit"; inspect with `trace.compiler.ir.result`. "Because the verifiable fragment of Lean cannot create cyclic data, the Lean runtime does not have a technique to detect it."
- Release 4.30.0 [S22, P]: "completion of the new LCNF compiler backend with user-controllable borrow annotations"; "#12781 ports the C emission pass from IR to LCNF, marking the final step of the IR/LCNF conversion".
- IR design paper [S23, P]: "Counting Immutable Beans" describes λpure → λRC with explicit inc/dec/reuse.
- Zulip thread [S58, S]: C backend and LLVM backend both exist for IR; "Right now, even with the LLVM backend enabled, lean still compiles _itself_ using the C backend" (thread date not stated in excerpt; may be out of date).

### 4.2 What is and is not trusted (as stated)

- Axioms page of the Lean reference [S17, P]: "Any proof that relies on an axiom can be trusted only to the extent that the axiom is both true and consistent with the other axioms used." "The `native_decide` tactic creates a bespoke axiom for each invocation". Native-evaluation axioms track proofs depending on "the correctness of the entire compiler, and not just on the much smaller `kernel`." The page does not define a "trusted base".
- Lean4Lean [S18, P]: "Lean4Lean is an external verifier for Lean, written in Lean" that checks Lean, Batteries and Mathlib; "Currently, only a few functions from the typechecker have been verified, like inferLambda."; the C++ kernel "was one of the few components that was not rewritten". It reports "at least one soundness bug" found during verification effort. It does not support `reduceBool` because general behaviour is "unsound by design": "intermediate-representation code can call external C functions, and `implemented_by` can separate compiler behavior from the kernel's implementation" (paraphrase of the fetch).
- Itemised trust list [S19, S]: "Trusting the correctness of compiled Lean programs requires trust in Lean's compiler, which is separate from the kernel and is not part of Lean's core logic." Items: (1) "any compiler overrides (extern / implemented_by) do not violate the lean logic (i.e. the model matches the implementation)"; (2) "the lean compiler (which lowered the lean code to C) to preserve the semantics of the program"; (3) "clang / LLVM to convert the C program into an executable with the same semantics".
- Community thread [S20, S]: "Use of `reduceBool` increases the trustbase considerably, including to every single `implemented_by` in any dependency"; fixed by adding a `trustCompiler` axiom (later replaced, per another blog, S, https://noferi.it/en/blog/dieci-dimostrazioni-certificate/, by a per-invocation axiom since Lean 4.29.0).
- Official claim on TCB [S15, P]: "requiring trust only in Lean's minimal proof-checking kernel" (context: checking proofs, Cedar).
- Slide claim [S29, P]: for SampCert, "Custom code extractor: the deployed artifact does not go through the Lean compiler. TCB includes the extractor, not the compiler."
- Claims in a third-party repo [S51, S, snippet only]: `turbo-lean` states it contains a "formally verified native compiler (`LeanNative`)" over an LNIR; its own table lists as TRUSTED "Adequacy of the LCNF/LNIR operational model … with respect to the Lean C runtime (`lean.h`)". Not evaluated here.

### 4.3 FFI and `extern`

[S16, P] Lean reference, Foreign Function Interface:
- `@[extern "sym"]` "causes compiled code to use the C declaration `sym` as the implementation."
- ABI "based on the standard C ABI and calling convention of the target platform"; `UInt8..UInt64`→`uint8_t..uint64_t`, `Float`→`double`, `Nat`/`Int`→`lean_object *`.
- "The current interface was designed for internal use in Lean and should be considered unstable."
- Borrow annotations: "From the perspective of Lean's type system, this annotation has no effect."
- FFI "currently limited to transferring Lean data types; compound data structures such as C `struct`s cannot yet be passed or returned by value" (fetch summary).
- The page "makes no general statement about the correctness of the compiler or runtime."

Examples of FFI-boundary use:
- lean-zip [S46, S]: "The native `def`s are the implementations being verified, the `Spec/` theorems are the specs, and `@[extern]` opaques are trusted." C code in `c/zlib_ffi.c` has "no Lean body".
- LeanPort [S45, P, snippet only]: "checks the proofs, compiles the Lean implementation to C, packages a native static archive, and generates the C/cgo bridge plus an idiomatic Go API" (Lean code consumed from Go; direction is Lean → Go, not Go → Lean).

---

## 5. Lean as a program verifier

### 5.1 Std.Do / mvcgen / vcgen (Lean core)

- Introduced in Lean 4.22.0 (2025-08-14) [S26, P]: "a Hoare logic for monadic programs in `Std.Do.Triple`", tactic `mspec`, and `mvcgen` "to turn a Hoare triple proof obligation `⦃P⦄ prog ⦃Q⦄` into pure verification conditions". `SPred` and a proof mode for stateful assertions.
- Reference [S25, P]: `mvcgen` "breaks down a goal involving a program written using Lean's imperative `do` notation into a number of smaller verification conditions". Method: weakest-precondition (predicate transformer) semantics, specification lemmas (`@[spec]`), user-supplied loop invariants ("loops require a specified loop invariant"), leftover VCs closed by `grind`/`simp`/proof mode.
- Status change [S25, S26, P]: reference manual banner: "The `mvcgen` tactic and its supporting libraries have been superseded by `vcgen`, which supports a wider variety of proofs and performs better." Lean 4.35.0 (2026-10-12): "`vcgen`, a faster and more general successor of `mvcgen`"; "mvcgen is now deprecated". Release 4.35 notes also show `ensures`/`invariant`/`decreasing` annotations on `def`/`while` (docs excerpt). `vcgen` "proves properties of any kind of program that has a weakest precondition semantics, from monadic computations to the syntax trees of a small imperative language" (https://github.com/leanprover/downstream-lean4/blob/master/reference-manual/Tutorial/VCGen.lean, snippet only).
- Lean 4.31.0 notes [S26, P]: assertion language generalised "from `SPred` to any `CompleteLattice`"; experimental `mvcgen'`; "It can outperform `mvcgen` by a factor of over 100x on some synthetic benchmarks".
- Scope as documented: programs are Lean programs (shallow embedding in monads such as `Id`, `StateM`, `ExceptT`, `IO`-like stacks). The verified object is the Lean code, not code in another language.

### 5.2 Loom and Velvet (verse-lab)

- Loom paper [S27, P] (PACMPL, POPL 2026): "Loom is a framework for building foundational, multi-modal program verifiers" using monad transformers and "monad transformer algebras to derive sound verification-condition generators". "The entire development of this paper comes with end-to-end correctness proofs done in Lean." Effects: `StateT`, `ReaderT`, `ExceptT`, `NonDetT`, `DivM`, `Gen`. Modes: testing, symbolic execution, deductive proofs; SMT (Z3, cvc5) plus Lean tactics. Two instances: Velvet (Dafny-style imperative verifier) and Veil 2.0 (distributed-protocol safety, see §12).
- Stated limitations [S27]: not all monadic representations of an effect are covered; new effects need supplied algebra instances; no "clever strategies for efficient constrained input generation" for Velvet's property-based testing; SMT interface lacks native Lean arrays (uses an `arrInt` interface); future work includes interaction trees and separation logic. Velvet case studies in the paper: 10 (insertion sort, linear/binary search, sqrt, cbrt, sparse-matrix computations).
- Velvet [S28, P]: "a Dafny-style verifier for imperative programs embedded in the Lean proof assistant"; `requires`/`ensures`/`invariant` macros; "Programs are shallowly embedded in a monad. `Id`, `Option`, `StateT`, `ExceptT` and stacks thereof are supported." A third-party list (https://quint.sh/guides/formal-verification-tools, S, snippet only) labels it "Research prototype, published at CAV 2026".
- Talk slide [S29, P]: "Velvet (V. Gladshtein): previously ~3× slower than Dafny; after the port, faster on 24 of 27 benchmarks. Dafny trusts its VC generator and an SMT solver; Velvet's chain is checked by the kernel." (https://leodemoura.github.io/static/marktoberdorf2026/lecture4/index.html, snippet only.)
- Velvet as a back-end for translated code: LemmaScript emits Velvet 2 definitions from TypeScript (§7).

### 5.3 Other Lean-embedded verifiers (small / experimental)

- `lean-program-verifier` / `MiniVerify` (https://github.com/rahul-mahadik/lean-program-verifier, P, snippet only): "An imperative-program verifier in Lean … contains a proved verification-condition generator and an executable interpreter."
- `lean-surfaces` [S43, P, snippet only]: "Each language gets a **deep embedding**: the program's real AST becomes a Lean value, and a definitional interpreter gives it meaning" for "Python / C++ / Rust / SystemVerilog"; includes running "a whole Python file under the Lean semantics" and comparing with CPython. Maturity not stated in excerpt.
- Imp/deep-embedding vs shallow-embedding framing [S29-adjacent, P, snippet only] (https://leodemoura.github.io/static/marktoberdorf2026/lecture4/index.html): "Imp was a **deep** embedding: programs are data, semantics is a relation. Shallow alternative: programs are ordinary monadic Lean code … the style of mvcgen, Velvet, and Aeneas."

---

## 6. CompCert-style (verified compilation) and Lean

- CompCert [S47, P]: "the executable code it produces is proved to behave exactly as specified by the semantics of the source C program"; written in Coq/Rocq; target ARM, PowerPC, RISC-V, x86. Theorem form: "if the compiler, applied to the source S, produces the code C, without reporting a compile-time error, then the observable behavior of C improves on one of the allowed observable behaviors of S." The proof starts after "preprocessing, parsing, type-checking and elaboration" and ends before "assembling and linking". "complete, end-to-end formal verification has not been achieved yet: … about 90% of the compiler's algorithms … are proved correct …, but the remaining 10% (including elaboration, presimplifications, assembling and linking) are not verified."
- CompCert TCB [S48, P, snippet only]: a dedicated paper "The Trusted Computing Base of the CompCert Verified Compiler" (Bedin França, Blazy, Leroy et al.; https://ar5iv.labs.arxiv.org/html/2201.10280): guarantee holds "if the compiler succeeded in compiling a C program and that program executes with no undefined behavior".
- CompCert is in Coq/Rocq, not Lean. Lean-side items found:
  - `leancompcert` [S49, P, snippet only, "Experimental"]: "It does **not** claim end-to-end verified compilation from Lean. When generated C is accepted by CompCert, only the C-to-assembly portion receives CompCert's semantic-preservation guarantee". Lean proof covers "the current scalar, straight-line core".
  - Verified Lean-written compilers for toy languages: `karipov/verified-compiler` (https://github.com/karipov/verified-compiler, P, snippet only) and a tutorial compiler in "From Zero to QED" (https://sdiehl.github.io/zero-to-qed/21_verified_programs.html, S, snippet only).
  - `ship-your-interpreter` (https://github.com/BasisResearch/ship-your-interpreter/, P, snippet only): Lean 4 with a Sail-derived RV64 model; theorems `compile_correct` for WHILE-to-RV64 compilers.
  - Verity (§8) proves its EDSL→IR→Yul compilation steps in Lean.
  - Lean-MLIR peephole verification (§3 / S50).
- No Lean 4 verified compiler for full C found in this pass (absence in the sources searched; not a claim that none exists).

---

## 7. Other source languages → Lean

### 7.1 Overview table

| Source language | Tool | Mechanism | Soundness basis as stated | Subset / features | Maturity as stated | Source |
|---|---|---|---|---|---|---|
| Rust | Charon + Aeneas; hax 0.4 `into lean` | Compiler-MIR extraction → functional translation | Formalised on paper (ICFP 2022/2024); tools themselves trusted in SymCrypt study | Safe, sequential; no raw pointers / interior mutability / concurrency | Production use at Microsoft (SymCrypt) | S1,S3,S5,S9 |
| Machine code (Armv8) | LNSym | Lean ISA model + symbolic simulator | Conformance testing of ISA model | AArch64 subset (unspecified) | "currently under development" | S14,S15 |
| .NET CIL (C#, F#) | LeanCIL | Extract CIL, check against Lean model | Explicit trust boundary | Integers, structs, arrays, selected calls; no exceptions/floats/virtual dispatch | Not stated | S37 |
| Yul / Solidity | Clear (Nethermind) | Yul → Lean embedding via Haskell VC generator | Lean proof of embedding consistency | Yul dialect model | Repo "archived" per S34/formal-eth listing | S33,S34 |
| Solidity-like EDSL | Verity | Write in Lean EDSL, compile to Yul with Lean proofs | Compiler layers proved; `solc` trusted | "supported fragment" | Active | S35 |
| EVM bytecode | EVMYulLean + refinement proofs | Lean semantics + proofs | Conformance tests (99.99% of Cancun tests) | Cancun EVM | Active | S36 |
| TypeScript | LemmaScript | TS + `//@` annotations → Dafny or Lean (Velvet 2) | Not a proof of translation; trust boundaries named per case study | Subset (see SPEC.md) | "Tech Preview" | S38 |
| TypeScript | Thales | Strict TS subset → Lean module | Not stated | No classes, mutation, async, `any`/`unknown` | Not stated | S39 |
| Python | PastaLean | Deterministic AST transpiler → "provable twin" + "runnable twin" | Differential testing; twins not proved equivalent | Small self-contained scripts; no `exec`, async, dynamic dispatch | Research | S40 |
| Python | code2lean | LLM translates + 5 validation gates | Differential tests + critic LLM | Function-level | Pipeline | S42 |
| C++, Go, Solidity, Rust | leanlift | LLM-proposed Lean + differential execution; Rust via Aeneas | "An LLM translation is never trusted" | Pure-ish functions over machine ints/structs | "preliminary … v0.1, draft" spec | S41 |
| Go (bounded) | go-in-lean | Hand model of `math/bits` in Lean, failed proofs → Go tests | Kernel + `bv_decide` on the model | Fixed-width ints | Proof of concept | S44 |
| C, Python (dataset) | StanfordLeanClub c-py-dataset (HF) | Synthetic translations | n/a | 107 records | Dataset | https://huggingface.co/datasets/StanfordLeanClub/c-py-dataset (S, snippet only) |
| C/C++ ... | "Skills" prompts (c-cpp-to-lean4, python-to-lean4) | LLM prompt instructions | None stated | n/a | Prompt text, not a tool | S55 |
| Move | Move Prover (Boogie/SMT, not Lean); Move borrow checker formalised in Lean | n/a (Lean used for checker metatheory) | Mechanised proofs about the checker | n/a | n/a | S29,S53 |

### 7.2 Per-tool notes

**Clear (Nethermind), Yul/Solidity** [S33, P]: "Clear extracts Yul smart contracts into an embedding in the Lean proof assistant, allowing users to write computer-checkable proofs of correctness for the extracted contracts." Three parts: a Lean model of the Yul EVM dialect, Lean tactics and theorems, and a verification-condition generator (Haskell, built with Stack). Inputs required: program, specification, proof. "It translates your program into an abstract representation and verifies the consistency of this translation." Users compile Solidity with special options so the Yul meets "structural assumptions of the framework" (S blog mirror, https://blockchain-development-solutions.com/en/blog/clear-formal-verification-framework-solidity). Status: another writeup lists "Nethermind's Clear, now archived" [S34, S]; a formal-eth list also marks it "(archived)" (https://github.com/NyxFoundation/formal-eth, S, snippet only).

**Verity** [S35, P]: Lean 4 EDSL where "Every contract is three things at once: a specification …, an implementation that runs, and a proof". FAQ: "Verified: the EDSL semantics, the `CompilationModel → IR` transformation for the supported fragment, the `IR → Yul` preservation, and per-contract specs and invariants. Trusted: the Lean kernel, the Yul compiler (`solc`), the EVM specification, the deployment toolchain, and the named external assumptions listed in `--trust-report`". README: "Write a contract in an embedded DSL (or import it from Solidity)"; "Yul → bytecode is done by pinned `solc` and is trusted." Another paper describes it as "mechanically verified compiler to Yul IR, which is then lowered to EVM bytecode by the unverified solc compiler" (https://ar5iv.labs.arxiv.org/html/2607.26306, snippet only).

**LemmaScript** [S38, P]: "ordinary TypeScript with `//@ ` specification annotations"; "generates verifiable code from your TypeScript — either in Dafny or Lean 4 (with Velvet 2)". Lean outputs: generated `.types.lean`, `.def.lean`; hand-written `.spec.lean`, `.proof.lean` (`prove_correct`). "Tech Preview: the core idea is there, but support, semantics, and ergonomics are still evolving." Union types LemmaScript cannot discriminate become an "opaque fall-through type" that "can only be passed through, never unsoundly observed." Case studies each name unverified boundaries, e.g. "The aggregate is proven; the React UI, WebSocket/DO I/O, and timezone labeling are the stated trust boundary" and "thin unverified adapter … differentially tested against real Python Guardians". A case study claims the verified function is "the proof target rather than a parallel model."

**Thales** [S39, P]: "A TypeScript-to-Lean 4 compiler … emits a Lean 4 sidecar alongside the input `.ts` file". "Every program Thales accepts is also accepted by `tsc --strict`". Translations: nullable unions → `Option`; `@throws` → `Except`; `@total` requires termination. "Currently out: classes, mutation, async, `any`/`unknown`/intersection".

**PastaLean** [S40, P]: figures: HumanEval+ 151/164 elaborated (92.1%), 7,007/7,007 tests matching CPython; LeetCode 2,391/2,589 (92.4%), 208,736/215,126 (97.0%) tests matching; LiveCodeBench 469/476 (98.5%). Statement: "We do not prove that the runnable twin refines the provable one." Unsupported statements can degrade to `pyUnsupported` placeholders. Out of scope: `exec`, monkey-patching, async/threads, dynamic dispatch, `super()`, metaclasses.

**leanlift** [S41, P, draft spec v0.1]: certification levels L0 typechecks, L1 conformant/N vectors, L2 bounded-proved, L3 proved. Quotes: "Coverage ≠ proof."; "L1 is *testing*; 'conformant/N' must never be reported as 'verified'."; "Not a soundness oracle for the LLM."; Rust path via Charon+Aeneas is described as "sound by construction". Out of scope for v1: stateful code, concurrency, unbounded I/O, FFI-heavy code, full stdlib semantics.

**code2lean** [S42, P, snippet only]: five gates: sanitizer, Lean compile, axiom allowlist (`propext`, `Classical.choice`, `Quot.sound`), differential test against original, second-LLM critic.

**FVSpec** [S52, P, snippet only]: benchmark; "2,772 of them (25%)" of 11,039 scraped Python property-based tests translated into 9,415 Lean 4 specifications with `Impl.lean` (computable port) and `Spec.lean` (`sorry` proofs).

**Go**: no sound Go→Lean translator found in this pass. Items found: leanlift's LLM-based path, `go-in-lean` (hand-written model of `math/bits`, failed proofs become Go tests) [S44], LeanPort (Lean→Go direction) [S45].

**C**: no maintained sound C→Lean extractor found in this pass. Items found: Scylla (C→safe Rust, then Charon/Aeneas) [S2]; Aeneas newsletters (2025) mention "libclang-based frontend" for Scylla; leanlift LLM path for C++; prompt "skills" [S55]; `lean-surfaces` deep embedding for C++ [S43]; Lean→C direction (§4) and Lean→CompCert experiment [S49].

**SampCert extraction** [S32, P, snippet only]: "Our simple probabilistic language enables straightforward extraction of Lean code while minimizing the trusted computing base." Deployment route also "involves a translation from SLang to Dafny … and then from Dafny to Python".

---

## 8. Model-vs-implementation gap: documented practices

### 8.1 Cedar (AWS): Lean model + Rust implementation + differential testing

- [S30, P] Cedar docs: "A formal model implemented in Lean"; "A production authorization engine written in Rust. We use only the safe subset of Rust"; "A differential testing engine that can test automatically that #1 and #2 have the same semantics, ensuring the properties proven about the model will also be true for the production implementation."
- [S30, P] paper: "we use *differential random testing* (DRT) … to show that the models match the production code". "testing that the model and production code agree on the outputs." Models cover evaluator, authorizer, validator; symbolic compiler model "in the process of being written".
- [S56, P] AWS blog: "We confirm that the code matches the model by using *differential random testing*. This involves generating millions of random inputs and checking that both model and code produce the same output on every input."
- [S31, P] Lean use-case page: Lean models "approximately 10 times smaller than the production code"; "Lean's fast execution time of only 5 microseconds per test case, compared to Rust's 7".
- [S29, P] slides: "~100M differential random tests nightly."
- [S46, S] "Correspondence between Lean spec and Rust code is established via differential random testing, not formal proof."
- [S29/ai.engineer talk summary, S, https://ai.engineer/talks/lRa9sPaMyy4-your-code-has-bugs-lean4-has-proofs]: "This checks the Rust implementation against the formal model on sampled inputs; it does not establish a deductive proof that the Rust code agrees with the model for every input."

### 8.2 Other documented fidelity mechanisms

- Conformance testing of ISA models (LNSym) [S14, S15].
- Known-answer vectors, hardware-intrinsic differential tests, and reviewing the Lean spec against standards text (SymCrypt) [S5].
- Rust↔Lean equivalence tests in `rust-core-models` [S12].
- Statement checking: "Lean checks the proof, not whether its statement captures your intended specification." (LeanCIL [S37]); "The guarantee is exactly the proved theorem, including its preconditions and quantifiers."
- Axiom audits: LeanCIL `leancil verify — kernel check, replay, axiom audit`; code2lean axiom allowlist; Verity `--trust-report`; reproducible-gate example in a shielded-pool writeup [S34, S]: "one command rebuilds them and fails on any unfinished proof, extra axiom or changed statement".
- Translation-validation framing [S48-adjacent, S, snippet only] (Cambridge R277 slides, https://www.cl.cam.ac.uk/teaching//2526/R277/slides-verified-compilation.pdf): "∀S, C, Comp(S) = OK(C) ⟹ S ≈ C" versus "Translation validation: If Comp(S) = OK(C) and Validate(S, C) = true".

### 8.3 Per-approach trust list, as each source states it

| Approach | Components the source says are trusted or unverified | Source |
|---|---|---|
| Aeneas/Lean (SymCrypt) | rustc + backends; Charon; Aeneas; Lean kernel; `native_decide`/`bv_decide`; Aeneas std axioms; hand-written models of unsupported functions; Lean specs vs standards | S5 |
| hax Lean | No TCB statement in release post; `sorry`-initialised theorems | S9 |
| Lean compiled code | Lean compiler, `extern`/`implemented_by` overrides, C compiler | S19 |
| LeanCIL | extractor, CIL model, verification machinery | S37 |
| Verity | Lean kernel, `solc`, EVM spec, deployment toolchain, named external assumptions | S35 |
| LemmaScript | per-case-study unverified boundaries (UI, I/O, adapters, clock, store) | S38 |
| Cedar | equivalence of model and Rust implementation is by differential testing | S30, S46 |
| PastaLean | translator code; twins unproved equivalent | S40 |
| CompCert (Coq, reference point) | unverified preprocessing, elaboration, assembling, linking; Coq TCB | S47, S48 |

---

## 9. Lean kernel and external checking (supporting context)

- Lean4Lean [S18, P]: independent checker in Lean, checks Mathlib; only some functions verified.
- Lean Kernel Arena (https://arena.lean-lang.org/checker/official/, P, snippet only): site comparing kernel implementations; "Soundness (rightfully rejected tests) 80 ✅ 0 ❌".
- `lean4lean`-related tools: Lean4Less and `lean2agda` translation (https://github.com/formalverification/lean2agda, S, snippet only) – exporting Lean proofs to other systems.

---

## 10. Language-feature coverage matrix (as documented)

Legend: Y = documented supported; N = documented unsupported; ? = not stated in sources read.

| Feature | Aeneas/Lean | hax→Lean | LeanCIL | Thales | PastaLean | LemmaScript | Std.Do/Velvet (Lean code) |
|---|---|---|---|---|---|---|---|
| Loops | Y (nested return/break limited) | Y (via Aeneas; while → recursion + measure per S11) | Y (termination shown) | ? (structural recursion) | Y | Y (`invariant`/`decreases`) | Y (invariant required) |
| Mutation / mutable refs | Y (via functional model, borrows) | Y | Y (checked memory) | N | Y (value semantics; optional heap) | Y | Y (state monads) |
| Generics / traits | Y with restrictions | same | typed dispatch | ? | N for dynamic dispatch | ? | Y (Lean) |
| Unsafe / raw pointers | N | N | checked memory model | n/a | n/a | n/a | n/a |
| Interior mutability | N | N | ? | n/a | n/a | ? | n/a |
| Concurrency | N | N | N | N (async) | N (async/threads) | ? | Non-determinism modelled in Loom, not threads |
| Exceptions / panics | Y (`Result` / `fail`) | Y (`RustM`) | N (no handlers) | Y (`@throws`→`Except`) | Y | ? | Y (`ExceptT`) |
| I/O, external calls | Axioms/opaque | Assumption files | N | ? | Limited | Trust boundary | `IO`-like monads |
| Floating point | ? | ? | N | ? | Y (`Float`/`Real` twins) | ? | Lean `Float` (binary64) |
| Dynamic dispatch (`dyn`) | N (Charon "not yet fully supported") | N | Y (typed managed dispatch) | N | N | ? | n/a |

---

## 11. Selected quantitative facts

- SymCrypt/Aeneas/Lean: 237 KLOC Lean, 16.7 KLOC Rust [S5].
- Cedar: ~10× smaller Lean model; ~100M nightly DRT cases [S29, S31].
- Verity: all three compiler layers "complete and machine-checked by `lake build`. Zero `sorry`." [S35 verification page].
- Move borrow-checker formalisation: "39,000 lines of mechanized metatheory in 27 working days, using Claude as an AI coding assistant" [S29].
- EVMYulLean: 99.99% of official Cancun execution tests [S36].
- PastaLean: coverage figures in §7.2 [S40].
- `mvcgen'`: ">100x" speed-up on "some synthetic benchmarks" [S26].
- Velvet: "faster on 24 of 27 benchmarks" vs Dafny after port (slide) [S29-related].

---

## 12. Mentions of protocols in these sources (recorded neutrally)

- LNSym [S15, P]: "LNSym uses Lean as a specification language to model the Arm instruction semantics and cryptographic protocols".
- Loom [S27, P]: instantiated in "one for distributed protocol safety" (Veil 2.0); "The Veil 2.0 randomized simulator's soundness was verified in Loom: it produces only traces admitted by the specification." Veil is the protocol-oriented verifier; details belong in files 01 and 06.
- Nethermind [S36, P]: Lean-based "protocol modeling" and verification work across Halo2, Plonky3, Circom, EVM/Yul. Quote: "The team applies hard formal methods, protocol modeling, and interactive theorem proving to provide strong correctness assurances for production cryptographic systems."
- ZK/Protocol-IR proposal (https://hackmd.io/@wonj/protocol-ir-positioning, S, snippet only): proposes "Protocol IR semantics formalized in Lean by translation to ArkLib" and proving "Translation faithfulness … as a Lean meta-theorem"; lists hax as a reference ("Its direction is fixed: existing Rust code → formal spec for verification.").
- Signal-protocol Lean/hax write-up (https://positive-intentions.com/docs/technical/signal-protocol-formal-verification/hax-lean/, S, snippet only): extraction "Translates it into Lean 4 (via hax's `into lean` backend)" and "Replaces cryptographic primitives with abstract stubs when `crypto-backend` is disabled".
- No source read in this file states that Lean 4 cannot verify protocols, or that it can verify protocol implementations end to end. This statement records only what the sources in this file contain.

---

## 13. Version / time notes

- Lean releases referenced: 4.22.0 (2025-08-14, Std.Do and `mvcgen` introduced), 4.25.0 (2025-11-14, `mvcgen invariants?`), 4.28.0 (2026-02-17), 4.30.0 (2026-05-26, LCNF backend complete), 4.31.0 (2026-06-13), 4.32.0 (2026-07-13), 4.35.0 (2026-10-12, `vcgen` introduced; `mvcgen` deprecated) [S22, S25, S26].
- hax 0.4 released ~2026-09-29 [S9].
- Aeneas is "tied to a particular version of Lean" [S6]; LemmaScript pins Lean 4.34.0 [S38].
- Search-index "date" fields on some pages differ from publication dates; only dates printed in the pages themselves are recorded here.

---

## Open Questions

1. Aeneas translation correctness: the README cites ICFP 2022 and ICFP 2024 papers but the abstract page read does not state an end-to-end theorem for the Rust→Lean emission. Which parts (LLBC semantics, translation, Lean pretty-printing, hax packaging) are covered by proofs, and which by trust?
2. hax Lean backend 0.4: no published TCB or independent validation was found. Does a document list which Rust constructs the Aeneas-based backend rejects versus silently models? Reference [23] ("independent analysis" of hax semantic gaps) in S11 was not identified.
3. Unsafe Rust, FFI and `async` code: Aeneas documents these as unsupported or modelled by hand. Is there a published coverage figure (percent of crates in a corpus that extract) for Charon/Aeneas on real-world code? None found in S4 or S11.
4. Rust protocol-style code (state machines, networking, async I/O, `tokio`): no source read reports extracting such code with Aeneas or hax. Are there examples?
5. C and Go: no maintained, sound extractor to Lean was found. Are there unpublished or recent projects (for example, Lean-based C semantics, Go semantics, LLVM IR semantics beyond Lean-MLIR peepholes) not captured by the searches used?
6. LNSym: supported instruction subset, size of verified programs, and what the conformance-test oracle is were not stated in the README. Are there published case-study results (e.g., SHA, AES, AWS-LC routines)?
7. Lean compiler: is there any published formal proof or translation-validation result for Lean's LCNF/IR/C pipeline? The `turbo-lean` repository claims a verified LNIR pipeline; its claims were not independently examined here.
8. `vcgen` replaces `mvcgen` as of 4.35.0. How do Loom/Velvet and downstream tools (hax `hax_mvcgen`) track that change? Are there documented migration issues?
9. Velvet publication status: sources conflict in venue labelling (Loom: POPL 2026 via PACMPL; Velvet: "CAV 2026" per one third-party list). Primary venue for Velvet not confirmed.
10. Differential testing (Cedar, PastaLean, leanlift) versus proof of model–implementation correspondence: are there published quantitative bug-finding results (bugs found by DRT in Cedar, divergences classified by leanlift) that characterise the gap?
11. Move: only the borrow-checker formalisation in Lean was found; no Lean-based verification of Move programs themselves (Move Prover uses Boogie/SMT). Confirm via the primary paper/repository of the Lean borrow-checker work.
12. Python → Lean: PastaLean covers small scripts; no source found for translating large Python protocol libraries. LeanPython (zkSecurity, plan only) targets an interpreter in Lean for the leanSpec consensus specification; its status beyond the plan was not read.
13. Several sources are LLM-skill/prompt files (S55) or draft specs (S41); none of them demonstrate a working translation. Are there empirical evaluations of LLM-based Lean translation faithfulness beyond PastaLean, FVSpec and Vero (https://ar5iv.labs.arxiv.org/html/2608.13522, snippet only)?
14. Clear is described as archived by two secondary sources; Nethermind's own pages read here do not state that. Primary confirmation of status needed.
15. Several URLs in this file were read via search snippets only (tagged). A second pass could fetch full pages for S6, S8, S10, S12, S13, S24, S32, S42-S45, S49-S54, S57.
