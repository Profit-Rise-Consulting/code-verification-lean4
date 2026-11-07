# REF-L06: Loom and Velvet (foundational program-verifier frameworks in Lean)

Access date for all sources: 2026-11-07. Source IDs refer to section 14.

## 1. Identity
| Field | Value |
|---|---|
| Subject (protocol or algorithm) | Loom: a framework for deriving weakest-precondition-based verifiers for shallowly embedded monadic Lean programs from "monad transformer algebras". Velvet: a Dafny-style verifier for imperative Lean programs (loops, local mutable state, nondeterminism, ghost state, partial/total correctness), originally built on Loom. Case studies: sorting, searching, square/cube roots, sparse matrix computations, VERINA problems, LeetCode-style problems, a segment tree, graph data structures. |
| Spec it targets (e.g. RFC 8446 section) | No external standard. Velvet benchmarks are VERINA problems compared with Dafny solutions (S3). |
| Organisation / authors | VERSE Lab, NUS. Loom POPL 2026: Vladimir Gladshtein, George Pîrlea, Qiyuan Zhao, Vitaly Kurin, Ilya Sergey (S1). Velvet CAV 2026: Vladimir Gladshtein, Vitaly Kurin, Yueyang Feng, Dipesh Kafle, George Pîrlea, Qiyuan Zhao, Ilya Sergey (S3). |
| Primary URLs | https://github.com/verse-lab/loom ; https://github.com/verse-lab/velvet ; Loom paper https://doi.org/10.1145/3776719 (PACMPL 10, POPL, Article 77; PDF https://ilyasergey.net/assets/pdf/papers/loom-popl26.pdf); Velvet paper https://doi.org/10.1007/978-3-032-32526-6_11 (CAV 2026, LNCS pp. 228-242) |
| Repo, commit SHA, last commit date, licence | Loom default branch `master` @ `78928abc9054b31d0bea85985496490baae95244`, 2026-02-21, Apache-2.0. Loom snapshot pinned by Veil: `f0ea9100bec2e48afad7193053d6a89db3d57335`, 2026-11-06 (not on `master`; fetched by SHA). Velvet `main` @ `b2a349a57021b6d4e24ab403365b404db1ce7e08`, 2026-11-05, Apache-2.0. Other Loom branches include `popl26-artifact` (`c582e97`). |
| Lean toolchain (or other tool and version) | Loom master: `leanprover/lean4:v4.24.0`, requires Mathlib, lean-auto, lean-smt (for SMT). Loom pinned snapshot: `v4.35.0`, no Mathlib. Velvet main: `v4.35.0`, only dependency `plausible`; `CaseStudies/` is a separate Lake package requiring Mathlib `v4.35.0`. Velvet CAV 2026 evaluation: Lean 4.24.0, cvc5 1.3.1, Z3 4.15.4, lean-auto `f62266d` (S3). |
| Maturity | Research prototypes. Velvet paper reports external use (first place in a VeHa 2025 competition category; a ChinaSoft 2025 contest task) (S3). Not AI-generated as far as can be determined. |

## 2. Claim as stated by the authors (verbatim)
- Loom (S1, abstract): "We present a framework that facilitates and streamlines construction of program verifiers that are both foundational and multi-modal." Conclusion: "With the help of two non-toy verifiers built on top of Loom, Veil (Sec. 7) and Velvet (Sec. 8), we have implemented, tested, and verified correctness of more than 25 case studies, combining SMT-powered automation with interactive Lean proofs."
- Loom (S1, sec. 8): "Velvet allows one to verify stateful, possibly non-terminating programs with non-deterministic choices, manipulating arrays and algebraic data types." "Using Velvet, we have verified 10 case studies: in-place insertion sort, linear and binary search for square and cube roots calculations, and multiple computations on sparse matrices."
- Velvet (S3, abstract): "We present Velvet—a Dafny-style verifier for imperative programs embedded in the Lean proof assistant. Like Dafny, Velvet supports reasoning about effectful programs featuring mutable state, loops, and non-determinism. Unlike Dafny, Velvet seamlessly combines automated SMT-based proofs with the interactive proof mode of the Lean proof assistant".
- Velvet README (S4): "Foundational verification. A proved method yields `<name>.spec`, a plain Lean theorem that Velvet reuses at call sites and that you can use in handwritten proofs."

## 3. What is actually formalised
Loom core (pinned snapshot `f0ea910`, S2):
- `Loom/MonadAlgebras/Defs.lean:89`: `class MAlgOrdered (m : Type u -> Type v) (l : outParam (Type u)) [Monad m] [CompleteLattice l]` (an ordered monad algebra from which `wp` is derived).
- `Loom/MonadAlgebras/WP/Basic.lean:27`: `def wp (c : m α) (post : α -> l) : l := liftM (fun x => post x) c |> MAlg.μ`.
- Instances for state, exceptions, nondeterminism (`NonDetT'`), divergence (`Control/Div.lean`), etc. The paper introduces "partial monad algebras" to handle divergence soundly (S1 sec. 4: "To resolve this unsoundness, we introduce partial monad algebras.").
- Velvet's total-correctness split as given in S1 sec. 8.1:
  ```lean
  lemma partial_total_split {α} : ∀ (c1 c2 : VelvetM α) (P : Prop) (Q : α → Prop),
    eraseEq c1 c2 → triplePartial P c1 Q → tripleTotal P c2 (λ _, True) → tripleTotal P c1 Q
  ```

Velvet at current `main` (S4): the VC generator under `Velvet/Core/VCGen/` (Driver, Solve, Frontend, Util) is a modified copy of Lean core's `Lean/Elab/Tactic/VCGen/*`; `diffs/*.diff` record the changes (e.g. `diffs/Driver.diff` header `--- a/Lean/Elab/Tactic/VCGen/Driver.lean +++ b/Velvet/Core/VCGen/Driver.lean`) and `generate_vcgen_diff` regenerates them "against the Lean toolchain's copy". Default automation is `velvet_vcgen ... with finish`, which "attempts to automatically prove the correctness of the method using Lean's `grind` tactic and arithmetic simplification" (`docs/doc.md:13`). Velvet main no longer depends on Loom (no `require` for it in `lakefile.toml`).

Example statement, `CaseStudies/LeetProof/BestTimeToBuyAndSellStock.lean:43-60, 168-169`:
```
method maxProfit (prices : Array Nat)
  requires valid: precondition prices
  ensures maximum_profit: postcondition prices result
  ...
      invariant minPrice_is_min: ∀ k : Nat, k < i → minPrice ≤ prices[k]!
      invariant best_upper_prefix: ∀ a b : Nat, a < b ∧ b < i → pairProfit prices a b ≤ best
      decreasing remaining: prices.size - i
...
prove_correct maxProfit by
  velvet_vcgen [maxProfit, postcondition] with try finish
```
plus 6 helper theorems (lines 76-145) proved interactively.

Sparse-matrix case study (S1 sec. 8.2): `method SpMSpV (spm: SpM) (spv: SpV) return (out: arrVal) ensures size out = size spm ensures ∀ i < spm.size, out[i] = ∑ j ∈ spv.size, spv[j] * spm[i][j]`, with a nondeterministic scheduler interleaving row iterations.

## 4. Property classes covered
| Class | Status | Evidence |
|---|---|---|
| Functional correctness | yes | Hoare triples / `requires`-`ensures` contracts for Lean programs (S1, S3, S4). |
| Safety invariant | yes (via Veil) | Loom underlies Veil 2.0 invariant checking (REF-L04). |
| Liveness | partial | Total correctness / termination via `decreasing` measures (S1, S4); no temporal liveness. |
| Refinement | partial | `eraseEq` relates programs differing only in annotations (S1); no data refinement to external code. |
| Symbolic security | no | |
| Computational security | no | |
| Parsing | no | |
| Constant-time | no | |
| Other | yes | Property-based testing of contracts (`#derive_tester_for`), angelic/demonic nondeterminism, incorrectness-style reachability (S1 sec. 6.1). |

## 5. Link to implementation code
Category: verified code is the Lean program itself (shallow embedding), executable via the Lean compiler.
- S4: "programs can be compiled, executed, validated using property-based testing, and formally verified within one unified environment." Ghost variables "are erased from the compiled program."
- No translation from or to Rust/C/Go/etc.; the guarantee covers the Lean source, with the Lean compiler trusted for execution.

## 6. Trusted computing base and soundness hygiene
Keyword counts (`rg -w`, incl. comments):
- Loom master (9,628 Lean lines in 38 tracked `.lean` files at `78928ab`, 11 of them under `CaseStudies/`): `sorry` 1, `admit` 2, `axiom` 2, `native_decide` 0, `implemented_by` 0, `extern` 0, `opaque` 1, `partial` 43. Important: `Loom/SMT.lean` on master declares `axiom trust_smt : ∀ (p : Prop), p` (line 215; docstring "We trust the SMT solver's judgment when it returns \"unsat\""), an inconsistent axiom applied as `mkApp (mkConst ``trust_smt) goalType` (line 249) to close goals after an SMT `unsat` answer. Also on master, `Loom/TacticAsync.lean:84` calls `mvarId.admit`, and `CaseStudies/Tactic.lean:144` warns "`loom_solve_async` uses sorry to admit all the solved goals for now".
- Loom pinned snapshot `f0ea910` (6,071 lines): `sorry` 1 (inside a comment, `Loom/MonadAlgebras/Instances/Basic.lean:44`), `admit` 0, `axiom` 2 (no `axiom` declaration found; `git grep "^axiom"` empty), `opaque` 6, `partial` 2. No SMT module in this snapshot.
- Velvet main (34,674 lines: `Velvet/` 14,534, `CaseStudies/` 20,127): `sorry` 3, all in comments or strings (`Velvet/Core/VCGen/Frontend.lean:358`, `Velvet/Frontend/VCReport.lean:52,71`; the latter reports VCs "admitted with sorry"); `axiom` 0; `native_decide` 22, in `Velvet/Examples/NonDet.lean` and similar example files as execution tests such as `example : (pickGreater 10).run = some 211 := by native_decide` (line 36); `opaque` 3; `partial` 83.
TCB: Lean kernel; Lean compiler for execution and `native_decide` tests; Mathlib for case studies; for the CAV-2026 Loom-based Velvet, SMT via lean-auto (only `isPrime` used cvc5, S3). On current main, automation is `grind`, which produces kernel-checked proofs.

## 7. Scale, effort, automation
- Loom paper (S1): Velvet, 10 case studies; Veil 2.0 "17 specifications of 15 different distributed protocols"; "more than 25 case studies" overall. For the sparse case study, prior work needed "300-400 LOC. However, in our case, the first layer of the proof required no manual reasoning at all."
- Velvet CAV 2026 (S3): 27 VERINA case studies, 618 lines (303 of them spec, invariants and proofs); MacBook Pro M4, 64 GB. `grind` discharged all VCs except `isPrime` (cvc5 via lean-auto, about 1 s); 21 fully automatic, 6 needing manual proofs (the same 6 as in Dafny); on average 3.67x slower than Dafny; all but 2 under 3 s and all under 45 s (outliers `findMajorityElement`, `mergeSorted`); 3.3x fewer total lines than Dafny, proofs 4.9x shorter; RQ3 `memAlloc` needed 15 lemmas / about 250 lines; `isNonPrime` 9 lines using Mathlib's `Nat.prime_def_le_sqrt`.
- Velvet repo `CaseStudies/`: `LeetProof/` (51 files), `SegmentTree/`, `DataStructures/Graph/`; 61 `.lean` files, 20,127 lines in total.
- Not built here (section 10).

## 8. Limitations stated by the authors (verbatim)
- S3: "The flip side of Velvet's design is that it does not currently support global heap state or OOP-style classes; programs that essentially rely on shared mutable references (e.g., doubly-linked lists with aliasing) are outside its present scope."
- S3, on Lean core: "Lean's own VC generator for monadic computations of the Std.Do library provides facilities for reasoning about imperative computations, but does not support programs with non-terminating loops or non-determinism."
- S3: "In the future, we plan to tackle multi-layer verification efforts of large-scale systems in the style of IronFleet [12,15] or Verdi [32] by making use of both Velvet and Veil in tandem."
- S1 footnote: "For now, we do not implement any clever strategies for efficient constrained input generation for a given precondition."
- S1 conclusion lists Separation Logic support as future work: "extending Loom to support a version of Separation Logic [17] on top of the state monad transformer."

## 9. Independent evidence
- S3 reports use outside the core team: "In November 2025, Velvet was used to win the first place in one of the categories of the regional VeHa verification competition" and memAlloc "was offered as one of the tasks in the 2025 CCF ChinaSoft Theorem Proving Contest." These are author-reported.
- A LeetProof paper using Velvet for certified program synthesis exists (arXiv 2604.16584), not read in full (section 15).
- No independent review or third-party bug reports were read.

## 10. Reproducibility
Not built: sandbox disk limit; queued for CI reproducibility job. The attempted `cd /tmp/velvet && timeout 1500 lake build` was stopped while elan was downloading `v4.35.0` after the shared sandbox ran out of disk; it was not retried, and no Loom build or `lake exe cache get` was run. Exact commands for CI:
```
git clone --depth 1 https://github.com/verse-lab/velvet && cd velvet && git checkout b2a349a57021b6d4e24ab403365b404db1ce7e08 && timeout 1500 lake build
(cd CaseStudies && lake exe cache get && timeout 3600 lake build)
git clone https://github.com/verse-lab/loom && cd loom && git checkout f0ea9100bec2e48afad7193053d6a89db3d57335 && timeout 1500 lake build
```
Loom POPL artefact: https://doi.org/10.5281/zenodo.17347734 (not downloaded).

## 11. Generalisation
Pattern: write the program directly as monadic Lean code; derive a weakest-precondition VC generator from a monad algebra (Loom) or from a fork of Lean core's `vcgen` (current Velvet); discharge VCs with `grind`/SMT and fall back to interactive proofs; test contracts before proving. Applying it to an arbitrary existing codebase requires rewriting that code in Lean (or translating it), since there is no front end for other languages; programs that need a global heap, aliasing or OOP classes are explicitly out of scope (S3). Preconditions: code that fits Lean's functional model with local mutation, arrays and ADTs.

## 12. Bearing on sub-claims (facts only, no verdict)
| Sub-claim | Fact from this reference | Source ID |
|---|---|---|
| H1 | Imperative programs with loops, nondeterminism, ghost state and arrays are expressible as Lean monadic code; heap/aliasing are not supported. | S1, S3 |
| H2 | 27 VERINA problems: 21 automatic, 6 manual, average 3.67x slower than Dafny, 3.3x fewer lines; 10 Loom-paper case studies including sparse matrices. | S1, S3 |
| H3 | Verified artefact is the Lean program itself, compiled by Lean; no link to code in other languages. | S3, S4 |
| H4 | Authors exclude global heap, OOP classes and aliasing; S2 (REF-L04) adds that C-like memory models may not suit shallow embedding. | S3 |
| H5 | Contracts can be tested via generated executable checkers before proving. | S1, S4 |
| D | "Foundational" = VC generator proved sound in Lean; "multi-modal" = automated plus interactive proofs and testing in one tool. | S1, S3 |

## 13. Conflicts
- Velvet's basis: S1 and S3 describe Velvet as built on Loom (S3: "Veil [27] is another multi-modal verifier built on top of Loom"), but Velvet `main` at `b2a349a` has no Loom dependency and uses a modified copy of Lean core's `vcgen` with `grind`.
- Automation: S3 abstract says Velvet "combines automated SMT-based proofs", while its evaluation used `grind` for all but one case and the current README says "SMT-inspired proof automation".
- Loom versions: master (v4.24, Mathlib, `axiom trust_smt`) and the snapshot Veil pins (v4.35, no Mathlib, no SMT module, no axiom declaration) differ materially; statements about "Loom" depend on which is meant.
- Case-study counts: S1 reports 10 Velvet case studies; S3 reports 27 VERINA case studies; the repo's `CaseStudies/` has 61 files.

## 14. Sources
| ID | Title | URL | P/S | Depth | Summary |
|---|---|---|---|---|---|
| S1 | Foundational Multi-Modal Program Verifiers (Loom, POPL 2026) | https://dl.acm.org/doi/10.1145/3776719 (PDF https://ilyasergey.net/assets/pdf/papers/loom-popl26.pdf) | P | FULL | Monad transformer algebras, WP derivation, partial algebras for divergence; Veil 2.0 (sec. 7) and Velvet (sec. 8). Sections 4-6 read more lightly than 1-3 and 7-10. |
| S2 | verse-lab/loom @ 78928ab (master) and @ f0ea910 (Veil pin) | https://github.com/verse-lab/loom | P | CODE | Definitions, toolchains, `trust_smt` axiom on master, counts. |
| S3 | Velvet: A Foundational Multi-modal Verifier for Imperative Programs in Lean (CAV 2026) | https://doi.org/10.1007/978-3-032-32526-6_11 | P | FULL | Velvet design, 27-problem comparison with Dafny, limitations, adoption. |
| S4 | verse-lab/velvet @ b2a349a | https://github.com/verse-lab/velvet | P | CODE | Current implementation (forked Lean vcgen, grind), case studies, counts, README claims. |

## 15. Unverified leads (excluded from evidence)
- LeetProof / "Certified Program Synthesis with a Multi-Modal Verifier", https://arxiv.org/html/2604.16584v1 (search result only).
- Velvet website https://velvetprover.dev and tutorial repo https://github.com/verse-lab/velvet-tutorial (not read).
- A third-party repo named lean4-velvet, https://github.com/metroncorp/lean4-velvet (search result; relation unknown).
- NUS news report that the Velvet paper received a CAV 2026 distinguished-paper award (seen in a search snippet, not read).
- Loom POPL artefact, https://doi.org/10.5281/zenodo.17347734 (not downloaded).
