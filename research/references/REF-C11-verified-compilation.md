# REF-C11: Verified-compilation boundary (CompCert, CakeML, and Lean's compiler)

Kind: comparator (non-Lean). Access date for all sources: 2026-11-07. Scope: a short boundary dossier on what a verified-compiler theorem transfers to a binary and what stays trusted. It is not a history of compiler verification.

## 1. Identity
| Field | Value |
|---|---|
| Subject | Compiler-correctness theorems as a link from verified source to executable: CompCert (C → assembly), CakeML (ML → machine code), and, for contrast, Lean 4's own compiler (Lean → C → native) |
| Spec it targets | CompCert: the CompCert C semantics `Csem.semantics` versus the per-architecture `Asm.semantics` [C11-S2]. CakeML: the CakeML source semantics versus the `machine_sem` of the target ISA model [C11-S4, C11-S5]. Lean: none (no compiler-correctness specification found in the sources read) [C11-S6, C11-S7]. |
| Organisation / authors | CompCert: Inria / AbsInt (X. Leroy et al.) [C11-S1]. CakeML: Tan, Myreen, Kumar, Fox, Owens, Norrish and the CakeML team [C11-S4]. TCB analysis: Monniaux and Boulmé (VERIMAG) [C11-S3]. Lean: Lean FRO (reference manual) [C11-S6, C11-S7]. |
| Primary URLs | https://compcert.org/man/manual001.html ; https://github.com/AbsInt/CompCert ; https://arxiv.org/abs/2201.10280 ; https://cakeml.org/jfp19.pdf ; https://github.com/CakeML/cakeml ; https://lean-lang.org/doc/reference/stable/ValidatingProofs/ |
| Repo, commit SHA, last commit date, licence | AbsInt/CompCert `2685140bf72c959446981fb99b5e44a8975021bc`, 2026-11-06; root `LICENSE` file present, GitHub reports no SPDX id (file not read). CakeML/cakeml `79f649eae40e37b71f1dd9d8dcaebfa5b8475cb8`, 2026-11-06, BSD-3-Clause. |
| Tool and version | CompCert: Rocq/Coq, proofs extracted to OCaml [C11-S1, C11-S3]. CakeML: HOL4, with the compiler bootstrapped in the logic [C11-S4]. Lean (contrast): Lean 4 stable reference manual as of v4.35.0 [C11-S6]. |
| Maturity | CompCert: production (commercial releases by AbsInt [C11-S3]). CakeML: research. |
| AI provenance | commit-trailers. Basis: GitHub API, last 100 commits, checked 2026-11-07. CakeML/cakeml has 83/100 commits carrying `Assisted-by: Codex <noreply@openai.com>` or `Bot: sol (OpenAI GPT-6.1 Sol)` lines (2026-10-27 to 2026-11-06). AbsInt/CompCert has 0/100 (range back to 2026-05-19). No AGENTS.md/CLAUDE.md-type files at either repo root. The cited theorems and the JFP 2019 paper predate these commits. Recorded as provenance only, not as a discount on the mathematics. |
| Evidence level | static-inspected (theorem statements read at pinned commits; not rebuilt). Bug-history statements: author-reported in [C11-S3]. |

## 2. Claim as stated by the authors (verbatim)
- CompCert manual §1.2: "Semantic preservation theorem: For all source programs S and compiler-generated code C, if the compiler, applied to the source S, produces the code C, without reporting a compile-time error, then the observable behavior of C improves on one of the allowed observable behaviors of S." [C11-S1]
- CompCert manual §1.2: "this theorem has been proved, with the help of the Rocq/Coq proof assistant, taking S to be abstract syntax trees for the CompCert C language (after preprocessing, parsing, type-checking and elaboration), and C to be abstract syntax trees for the assembly-level Asm language (before assembling and linking)." [C11-S1]
- CakeML JFP abstract: "The compiler produces machine code for five architectures: x86-64, ARMv6, ARMv8, MIPS-64, and RISC-V. The generated machine code contains the verified runtime system which includes a verified generational copying garbage collector and a verified arbitrary precision arithmetic (bignum) library." [C11-S4]
- CakeML JFP §11: "Bootstrapping removes the need to trust an unverified code generation process. By contrast, CompCert first relies on Coq's extraction mechanism to produce OCaml, and then relies on the OCaml compiler to produce machine code." [C11-S4]

## 3. What is actually formalised
- CompCert proves backward simulation from CompCert C abstract syntax to Asm abstract syntax for the verified phase ("8 intermediate languages and 15 compilation passes") and also verifies the parser. Preprocessing, elaboration, presimplifications, textual assembly printing, assembling and linking are outside the proof [C11-S1 §1.3]. Separate compilation and linking at the CompCert C / Asm level are covered since version 2.7 (`separate_transf_c_program_preservation`, Complements.v:244) [C11-S2, C11-S3 §5.4].
- CakeML proves a top-level theorem (`compile_correct_gen`) from the source semantics to `machine_sem` of a target machine model. It covers code generation and assembly to bytes, and includes the garbage collector and bignum library [C11-S4, C11-S5]. The program-specific assembly wrapper and the linking/loading step are not verified: "compiler ui is not verified in the same sense ... Verifying it would require verifying the small assembly wrapper ... This would require formalising the linking system, which we have not done." [C11-S4 §10]
- Lean 4: the compiler translates the stored IR "into C code. A C file is produced for each Lean module; these are then compiled to native code using a bundled C compiler." [C11-S7 §2.4]. The sources read state no correctness theorem for this pipeline. The reference manual places the compiler outside the kernel's trust boundary, and says that native evaluation "depends on the correctness of the Lean compiler and all definitions with an `@[implemented_by]` attribute." [C11-S6]

## 3a. Predicate semantics

CompCert, `driver/Complements.v:31-35` @ 2685140 [C11-S2]:
```
Theorem transf_c_program_preservation:
  forall p tp beh,
  transf_c_program p = OK tp ->
  program_behaves (Asm.semantics tp) beh ->
  exists beh', program_behaves (Csem.semantics p) beh' /\ behavior_improves beh' beh.
```
Corollary `transf_c_program_is_refinement` (lines 45-49) adds the premise `(forall beh, program_behaves (Csem.semantics p) beh -> not_wrong beh)` and concludes that every Asm behaviour is a source behaviour. Conclusion predicate, `common/Behaviors.v:81-82`:
```
Definition behavior_improves (beh1 beh2: program_behavior) : Prop :=
  beh1 = beh2 \/ exists t, beh1 = Goes_wrong t /\ behavior_prefix t beh2.
```
`program_behavior` (lines 41-45) is `Terminates trace int | Diverges trace | Reacts traceinf | Goes_wrong trace`. Observable events exclude "execution time and memory consumption" [C11-S1 §1.2].

Plain language (CompCert). Guaranteed: if compilation succeeds, each behaviour of the Asm AST, under CompCert's Asm semantics, equals a source behaviour, or extends one whose source run had undefined behaviour. If the source has no undefined behaviour, Asm behaviours are a subset of source behaviours. Not guaranteed: properties of the textual assembly, the object file or the linked binary; agreement of `Asm.semantics` with the real ISA and ABI; timing or memory use; and source programs that have undefined behaviour.

CakeML, `compiler/proofs/compilerProofScript.sml:202-224` @ 79f649e (success case) [C11-S5]:
```
    | M_success (code,data,c) =>
      ∃behaviours source_decs.
        (semantics st prelude input = Execute behaviours) ∧
        parse (lexer_fun input) = SOME source_decs ∧
        ∀ms.
          installed code cbspace data data_sp c.lab_conf.ffi_names
            (heap_regs cc.backend_config.stack_conf.reg_names) mc c.lab_conf.shmem_extra ms
            ⇒
            machine_sem mc st.sem_st.ffi ms ⊆
              extend_with_resource_limit'
                (∀po. is_safe_for_space (:α) st.sem_st.ffi po cc
                        (prelude ++ source_decs) (read_limits cc mc ms))
                (semantics_prog st.sem_st st.sem_env (prelude ++ source_decs))
```
The premise is `initial_condition st cc mc` (lines 28-47), which includes `backend_config_ok`, `mc_conf_ok mc` and `mc_init_ok`. The failure cases `AssembleError`, `ConfigError` and `StaticError` conclude only `T` (lines 209-211). `installed`, per [C11-S4 §10.1]: "checks that the machine state ms, and machine configuration mc are set up correctly with respect to the output code, the data section, and the FFI ... Each of these assumptions turns into a corresponding part of the (unverified) wrapper assembly around our generated bytes."

Plain language (CakeML). Guaranteed: if compilation succeeds and the machine starts in an `installed` state, every machine-level behaviour is a source behaviour, except that the program may stop early with a resource-limit outcome. This holds relative to the FFI oracle and an environment-interference oracle that models the OS [C11-S4 §9.5]. Not guaranteed: that the unverified wrapper, the loader or the OS actually establish `installed`; agreement of the ISA model with silicon; FFI implementations; and anything when compilation fails.

Lean 4 (contrast): no compiler-correctness predicate exists to quote. The nearest formal object is the per-computation native-evaluation axiom. The reference manual says: "If axioms with `_native` in their names are reported, then native evaluation is used." [C11-S6 §Printing Axioms]

## 4. Property classes covered
Compiler functional correctness (semantic preservation): proved for CompCert C AST → Asm AST and for CakeML source → machine model. Preservation of source safety specifications: proved for CompCert (`transf_c_program_preserves_spec`, line 159; only for "safety-enforcing" specifications, `safety_enforcing_specification` at line 151) [C11-S2]. Constant-time or side-channel preservation: not covered by these theorems (time is not observable in [C11-S1 §1.2]). Lean compiler: none.

## 5. Link to implementation code
- CompCert: the compiler is extracted from Coq to OCaml and built with the OCaml compiler. The output is textual assembly, which an external toolchain assembles and links. "This toolchain is thus within the TCB." Valex (AbsInt) "matches the CompCert code to the binary code" for some architectures [C11-S3 §5.4; C11-S1 §1.3].
- CakeML: the compiler is bootstrapped in HOL4, producing verified machine code for the compiler itself. User programs are emitted as bytes inside an unverified per-target wrapper [C11-S4 §10, §11].
- What a compiler theorem transfers to a verified-source project (facts from the statements above): a source-level theorem about observable behaviours carries over to the compiler's target-semantics model of the output, under the theorem's premises (CompCert: no source undefined behaviour, and the property must be a safety-enforcing behaviour set; CakeML: `installed`, `initial_condition`, FFI oracle). It does not carry over to the linked binary on real hardware without the trusted steps listed in §6.

## 6. Trusted computing base and soundness hygiene
CompCert, per [C11-S3]:
- §5: "The verified parts of CompCert do not output machine code, let alone textual assembly code." Pseudo-instructions "are expanded by unverified OCaml". Printing goes through the `TargetPrinter` module.
- §5.4: the external assembler/linker is in the TCB.
- §6.3: "CompCert assumes that this allocator is correct with respect to CompCert's infinite memory model. In particular, CompCert assumes that malloc always succeeds and never returns the null pointer". Floating point assumes IEEE-754 round-to-nearest with "no change to the rounding mode". Runtime-library arithmetic helpers "are axiomatized".
- §2-3: the Coq kernel, extraction and OCaml are also in the TCB.
- §7: "None of the problems found were in the verified parts of CompCert", and "most bugs were found in the very last part of the back-end, which expands and prints assembly instructions."

CakeML, per [C11-S4]:
- unverified assembly wrapper and linking ("we have not done");
- the target ISA model;
- the FFI oracle and an "environment interference oracle ... to model the interference of an operating system" (§9.5);
- the HOL4 kernel.

Lean 4, per [C11-S6, C11-S7]:
- Kernel-checked theorems do not depend on the compiler unless native evaluation is used. With native evaluation, "for every `implemented_by`/`extern` attribute in libraries it becomes part of the trusted code base that the replacement is semantically equivalent" (§On `Lean.trustCompiler` (up to Lean 4.28.0)).
- "External checkers (`lean4checker`, `comparator`) cannot check such proofs, as they do not have access to the Lean compiler."
- "The `Lean.trustCompiler` machinery was removed from Lean in version 4.35.0."
- Executables additionally depend on the Lean → C translation and the "bundled C compiler" [C11-S7 §2.4]. No correctness theorem for either was found in the sources read.

Cross-references in this repository:
- research/06 E05 (Run-Time Code: reference counting, task runtime).
- E06 (`partial`/`unsafe`/`implemented_by`: "the behavior of programs written in Lean may diverge from their verified logical models if the unsafe implementation is incorrect").
- E07 (native-evaluation axioms "track proofs that depend on the correctness of the entire compiler").
- E08 (Validating a Lean Proof).
- V08 (4.29: one axiom per native computation; 4.34: refcount-overflow use-after-free fix #14838 and GMP ≥ 6.3.0).
- A01 (bv_decide TCB "the Lean kernel and the Lean compiler").
- Conflict C1 (size of the trusted base).
- research/03 E38/E39 (CompCert manual and TCB paper), E40 (lean-compcert), E41 (turbo-lean).

## 7. Scale, effort, automation
CompCert: "about 90% of the compiler's algorithms (including all optimizations and all code generation algorithms) are proved correct in Rocq/Coq, but the remaining 10% (including elaboration, presimplifications, assembling and linking) are not verified." [C11-S1 §1.2]. CakeML: 12 intermediate languages, five architectures [C11-S4 abstract]. No effort figures were collected for this short dossier.

## 8. Limitations stated by the authors (verbatim)
- "For phase 4 (assembly and linking), we have no formal guarantees yet, but the Valex tool, available from AbsInt, provides additional assurance via *a posteriori* validation of the executable produced by the external assembler and linker." [C11-S1 §1.3]
- "We emphasise that compiler ui is not verified in the same sense as the backend and toplevel compilers discussed above, and it does not have a corresponding correctness theorem." [C11-S4 §10]
- "In our opinion, the primary focus for increasing trust in CompCert (and removing possible further bugs) should be a validation mechanism of its assembly" [C11-S3 §7]

## 9. Independent evidence
[C11-S3] is a TCB analysis of CompCert by researchers who develop CompCert-KVX. They report that the bugs found were in unverified or specification parts, not in verified passes (§7). Its §5.4 mentions CompCertELF (a verified assembler for x86) and a verified ELF linker for a subset of x86-32. These were not read and are listed in §15.

## 10. Reproducibility
Not built: sandbox disk limit. The commands that would be used: `git clone https://github.com/AbsInt/CompCert && cd CompCert && git checkout 2685140bf72c && ./configure x86_64-linux && make` ; `git clone https://github.com/CakeML/cakeml && cd cakeml && git checkout 79f649eae40e && cd compiler/proofs && Holmake`. Files were read through the GitHub contents API at the pinned SHAs.

## 11. Generalisation
Both theorems connect two formal semantics, a source semantics and a target model. In both cases the last mile to a running binary (printing or wrapper, assembler or loader, linker, OS, hardware) stays outside the theorem, at different points: CompCert stops at the Asm AST, CakeML at bytes in an `installed` machine state.

## 12. Bearing on sub-claims (facts only, no verdict)
| Sub-claim | Fact |
|---|---|
| H3 | A source-level proof reaches a binary only through a compiler link. CompCert and CakeML supply a proved link down to their target models (§3a). Lean 4's shipped compiler (Lean → C → bundled C compiler) has no correctness theorem in the sources read [C11-S6, C11-S7]. |
| H5a / H5b | Any route that compiles verified Lean code to native code inherits the Lean compiler, the `implemented_by`/`extern` substitutions (research/06 E06) and the C compiler, unless a separate verified path is used. |
| D | "Verified" for CompCert means semantic preservation to the Asm AST. The manual itself says end-to-end verification "has not been achieved yet" [C11-S1 §1.2]. |
| Caveat (lean-compcert) | lean-compcert (research/03 E40) is not verification of Lean-to-native compilation. Its README describes "a Lean native backend and verified-computation package that obtains CompCert's verified C-to-assembly guarantee ... for its certificate fragment — not as a formally verified compiler of arbitrary Lean programs to machine code". Its `what-is-proved.md` says a "program" there "is not arbitrary Lean code and not arbitrary C" (a bounded register-machine fold). It inherits CompCert's assembler/linker/hardware assumptions (§6). Similarly, turbo-lean (E41) states refinement against an in-repo LCNF evaluator for a subset only, not against Lean's shipped compiler. |

## 13. Conflicts
- Wording of the CompCert guarantee. research/03 E38 quotes the manual as "the executable code it produces is proved to behave exactly as specified by the semantics of the source C program". The same manual says the theorem takes C to be "abstract syntax trees for the assembly-level Asm language (before assembling and linking)", and that assembly and linking have "no formal guarantees yet" [C11-S1 §1.2, §1.3]. Both statements are recorded without resolution.
- CakeML §11 [C11-S4] contrasts CakeML's in-logic bootstrapping with CompCert's reliance on extraction and OCaml. [C11-S3 §7] reports "no bugs due to the use of axioms for interfacing untrusted code, or the use of the extractor to OCaml, were found." Both are recorded.

## 14. Sources
| ID | Title | URL | P/S | Depth | Notes |
|---|---|---|---|---|---|
| C11-S1 | CompCert user's manual, chapter 1 "Overview" | https://compcert.org/man/manual001.html | [P] | [FULL] | Complete chapter text (≈28k chars) fetched 2026-11-07; §1.2, §1.3 quoted. |
| C11-S2 | AbsInt/CompCert @ 2685140 (2026-11-06): `driver/Complements.v`, `common/Behaviors.v` | https://github.com/AbsInt/CompCert | [P] | [CODE] | Theorem statements read; not built. |
| C11-S3 | Monniaux, Boulmé, "The Trusted Computing Base of the CompCert Verified Compiler", arXiv 2201.10280v2 (2022-10-10) | https://arxiv.org/abs/2201.10280 | [P] | [SECT] | Sections read: abstract, §5 (5.1-5.4), §6 (6.1, 6.3), §7. Verbatim: "This toolchain is thus within the TCB." |
| C11-S4 | Tan, Myreen, Kumar, Fox, Owens, Norrish, "The Verified CakeML Compiler Backend", JFP (preprint dated 2018-11-19) | https://cakeml.org/jfp19.pdf | [P] | [SECT] | Sections read: abstract, §2.4, §9.5-9.6, §10, §11. Verbatim: "This would require formalising the linking system, which we have not done." |
| C11-S5 | CakeML/cakeml @ 79f649e (2026-11-06): `compiler/proofs/compilerProofScript.sml` | https://github.com/CakeML/cakeml | [P] | [CODE] | Theorem statements read; not built. Commit history checked for AI provenance. |
| C11-S6 | Lean Language Reference, "Validating a Lean Proof" (stable, v4.35.0) | https://lean-lang.org/doc/reference/stable/ValidatingProofs/ | [P] | [FULL] | Complete page fetched 2026-11-07. |
| C11-S7 | Lean Language Reference, ch. 2 "Elaboration and Compilation" (stable) | https://lean-lang.org/doc/reference/stable/Elaboration-and-Compilation/ | [P] | [FULL] | Complete page HTML fetched (≈28k chars of text); §2.3, §2.4 quoted. |

## 15. Unverified leads (excluded from evidence)
- Lean 4.35.0 release notes (search snippet only): "#14953 removes `Lean.reduceBool`, `Lean.reduceNat`, `Lean.ofReduceBool`, `Lean.ofReduceNat` and `Lean.trustCompiler`". See research/06 V04 for the full-read entry. https://lean-lang.org/doc/reference/stable/releases/v4.35.0/
- CompCertELF (verified x86 assembler) and verified ELF linker for x86-32, cited in [C11-S3 §5.4]; not read.
- Lööw et al., CakeML on the verified Silver processor (hardware end of the chain); not read.
- Valex a-posteriori binary validation (AbsInt); product documentation not read.
