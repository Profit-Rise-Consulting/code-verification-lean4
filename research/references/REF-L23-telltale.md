# REF-L23: Telltale (asynchronous MPST choreographies: Lean 4 metatheory plus a Rust protocol-machine runtime compared against a Lean reference runner)

Source depth counts (section 14): 3 [FULL], 3 [SECT], 2 [CODE]. Evidence level: static-inspected (not built). AI provenance: none-found.

## 1. Identity
| Field | Value |
|---|---|
| Subject (protocol or algorithm) | Asynchronous buffered multiparty session types (MPST) and choreographies; a "protocol machine" (coroutine scheduler, sessions, buffered edges) that runs projected local types; a library of distributed-systems "theorem families" (FLP, CAP, Nakamoto, failure detectors, CRDT, etc.) |
| Spec it targets | No external standard. The specification is the Lean model itself: `SessionTypes`, `SessionCoTypes`, `Choreography`, `Protocol`, `Runtime` libraries (S1, S2) |
| Organisation / authors | Sam Hart (GitHub `hxrts`; papers list "Sam Hart Berman"). `LICENSE`: "Copyright (c) 2026 Sam Hart". Single author in the shallow history (`Sam Hart <sam@hxrts.com>`) |
| Primary URLs | https://github.com/hxrts/telltale ; papers `papers/paper1.tex` to `paper3.tex` in repo (also linked from README as https://hxrts.com/telltale/papers/paper1.pdf etc.) |
| Repo, commit SHA, last commit date, licence | hxrts/telltale @ `c8c6a7fb3700896d1ab31f4bb00e42de97682943`, 2026-10-31T21:52:06+01:00, MIT |
| Lean toolchain (or other tool and version) | `leanprover/lean4:v4.35.0` (root and `lean/lean-toolchain`). Lake manifest packages: mathlib, iris (iris-lean), paco, plausible, aesop, batteries, Qq, proofwidgets, LeanSearchClient, importGraph, Cli, leanfmt. Rust toolchain pinned in `rust-toolchain` |
| Maturity | Research (single-author active project with a public "formally verified for the declared shipped surface" claim, crates and release pipeline; no third-party users established) |
| AI provenance | none-found. No `AGENTS.md`, `CLAUDE.md`, `.claude/` or `.cursor/` present at this commit; no AI trailer on the head commit. Note only: `toolkit/toolkit.toml:43` and `:390-391` list `CLAUDE.md`/`AGENTS.md` as file-name exemptions in a lint configuration; these files do not exist in the tree. Not used to discount anything (CONVENTIONS). |

## 2. Claim as stated by the authors (verbatim)
- README (S1): "Telltale is formally verified for the declared shipped surface documented in docs/902_verification_inventory.md, under the listed public assumptions and trusted base."
- README (S1): "The claim covers the Lean semantics and theorems, the theorem-defined Rust↔Lean core protocol-machine runtime correspondence, the shipped first-party handler and transport contract boundary, and the shipped first-party crate artifacts through the audited artifact-correspondence pipeline. Compiler, macro, and third-party integration paths are excluded unless the verification inventory says otherwise."
- Verification inventory, "Final Public Claim Text" (S2): "The declared shipped surface includes the Lean semantics/theorems, the theorem-defined Rust↔Lean protocol-machine runtime correspondence core, the shipped first-party handler/transport contract boundary, and the shipped first-party crate artifacts through the audited artifact-correspondence pipeline. It does not include Rust compiler/macro entry paths or third-party integrations unless stated otherwise."
- Verification inventory, "Current Formal Verification Claim" (S2): "This does not mean every Rust API, compiler helper, macro path, plugin, or third-party integration is mechanically proved."
- Paper 1 abstract (`papers/paper1.tex:202-206`, read as abstract only, see §15): "We define an operational coherence invariant $\Coherent(G,D)$ over local-type environment $G$ and buffered-trace environment $D$, and show that this invariant is sufficient for preservation in asynchronous semantics."

## 3. What is actually formalised
Three artefacts share one repo (S1):
1. A Lean 4 proof stack (`lean/`): 830 `.lean` files, 164,896 lines, 5,105 `theorem`/`lemma` declarations (lexical count over all of `lean/` excluding `.lake`; includes tests and runners). `CODE_MAP.md:48` reports a smaller total, 767 files and 152,326 lines, under its own counting scope.
2. A Rust implementation (`rust/`, 234,015 lines across all `.rs` files, lexical): the `tell!` choreography macro, parser/projection, a protocol machine (`rust/machine`), threaded runtime, transports, simulator and a bridge crate (`rust/bridge`) that drives a Lean-built executable.
3. Three papers (`papers/paper1-3.tex`): coherence metatheory, Lyapunov/decidability results, "harmony" for reconfiguration. Not read beyond the paper 1 abstract (§15).

What the Lean side formalises (from code read, S3, S4, S6):
- A typed asynchronous session calculus: local-type environment `G : GEnv`, buffered-trace environment `D : DEnv`, an edge-wise coherence invariant `Coherent G D`, and send/recv step specifications `SendSpec`/`RecvSpec` with preservation theorems.
- A protocol-machine model: `ProtocolMachineState` with coroutines, sessions and a scheduler; `schedule`, `execInstr`, `updateAfterStep`; a canonical runner `runScheduled` and a threaded runner `runScheduledThreaded` (`Runtime/.../ThreadedRunner.lean:427`).
- A "concrete refinement slice" (`Runtime/Proofs/ProtocolMachine/ConcreteRefinement.lean`, 430 lines). Module doc (verbatim): the slice "intentionally tracks only: session-local type counts, buffered message counts, scheduler ready / blocked state, coroutine-local program counters and status tags. This is the smallest nontrivial state surface that the Rust runtime, Lean runner, and threaded runtime can compare exactly today."
- Distributed "theorem families" (`lean/Distributed/`), each with a `Model`, an `Assumptions` structure and exported theorems; the family README classifies their completeness (§3a, item 5).

How the Rust runtime is linked (S2, S5): the Rust bridge crate runs `lean/.lake/build/bin/protocol_machine_runner` (a Lean executable) over JSON on stdin/stdout and compares its per-step output with Rust executions on a corpus of fixtures (`rust/bridge/tests/protocol_machine_differential_steps.rs`, 558 lines; tests `tier1_step_indexed_correspondence` at line 535 and `tier2_step_indexed_correspondence` at line 549). The comparison logic includes two normalisation rules that the inventory classes as "irreducible trusted comparison logic" (S2, "Bridge Normalization Trust Surface").

## 3a. Predicate semantics
All file:line references are into `lean/` at `c8c6a7f` unless stated.

1. Runtime-correspondence headline: `project_concrete_scheduled_step_claimed_surface_spec` (`Runtime/Proofs/ProtocolMachine/ConcreteRefinement.lean:333`).
   - Definition used (`ConcreteRefinement.lean:88-95`, verbatim):
     ```
     def projectConcreteScheduledStep?
         (st : ProtocolMachineState ι γ π ε ν) : Option ConcreteScheduledStep :=
       match schedule st with
       | none => none
       | some (cid, stSched) =>
           let (stExec, res) := execInstr stSched cid
           let sched' := updateAfterStep stExec.sched cid res.status
           let stNext : ProtocolMachineState ι γ π ε ν := { stExec with sched := sched' }
           some (mkConcreteScheduledStep st.clock st stNext cid res.status res.event)
     ```
   - Conclusion (`:335-345`, verbatim excerpt): `step.preState = projectConcreteProtocolMachineSlice st ∧ step.postState = projectConcreteProtocolMachineSlice stNext ∧ step.transition = projectConcreteTransitionSlice stNext cid res.status ∧ step.event = projectConcreteTraceEvent? st.clock cid res.status res.event`, with `| none, none => True` and `| _, _ => False`.
   - Premises: none beyond the state `st`. Proof: `unfold` + `cases` + `simp` (`:346-352`). The four `mk_concrete_scheduled_step_*` lemmas (`:355-385`) are proved by `rfl`.
   - Plain language: this theorem states that the Lean-side "claimed transition object" is by definition the projection of one Lean scheduler step. It defines the surface Rust is compared against. It contains no Rust term; the Rust↔Lean equality is established by the bridge tests on a fixture corpus, not by a Lean theorem. Inventory TCB row (S2, verbatim): "| Rust runtime implementation | shipped executable semantics are still comparison-checked, not fully proved | strict correspondence, semantic assurance, refinement slice |".

2. Coherence preservation: `concrete_send_slice_preserves_coherent` (`ConcreteRefinement.lean:395`) and `concrete_recv_slice_preserves_coherent` (`:404`).
   - Conclusion predicate (`Protocol/Coherence/EdgeCoherenceCore.lean:94-95`, verbatim): `def Coherent (G : GEnv) (D : DEnv) : Prop := ∀ e, ActiveEdge G e → EdgeCoherent G D e`, where `EdgeCoherent` (`:74-84`) says that for every receiver local type `Lrecv` on the edge there is a sender local type and `(Consume e.sender Lrecv trace).isSome`, and `ActiveEdge` (`:89-91`) requires both endpoints present in `G`.
   - Strongest premises: `hSpec : SendSpec G G' D D' senderEp receiverRole T` (`Runtime/ProtocolMachine/Semantics/InstrSpec.lean:42-59`: sender has type `.send receiverRole T L'`, `G' = updateG G senderEp L'`, `D' = updateD D edge (lookupD D edge ++ [T])`, frame conditions on other endpoints and edges), `hCoh : Coherent G D`, and `hReady : SendReady G D` (`EdgeCoherenceCore.lean:359-364`: after consuming the current trace the receiver can consume one more message of type `T`).
   - Plain language: if a step satisfies the abstract send/recv specification and the pre-state is coherent (and for send, the receiver is ready), the post-state is coherent. That `execInstr` meets `SendSpec` on a particular instruction is a separate obligation not stated in these two theorems; whether it is proved elsewhere was not traced.

3. Scheduler determinism: `concrete_scheduler_slice_deterministic` (`ConcreteRefinement.lean:412`).
   - Predicate (`Runtime/Proofs/ProtocolMachine/SchedulerContracts.lean:30-38`): doc "`schedule` is a function, so its result is unique. This is determinism of coroutine selection, not confluence"; `def schedule_deterministic ... := ∀ st1 st2, schedule st = some st1 → schedule st = some st2 → st1 = st2`.
   - Plain language: true for any Lean function; it does not say that Rust's scheduler or the threaded runtime chooses the same coroutine. The same file documents `schedule_nonblocking` as "This is NOT fairness or starvation freedom".

4. Threaded refinement: `concrete_slice_threaded_one_refines_canonical` (`ConcreteRefinement.lean:422`): `runScheduledThreaded fuel 1 st = runScheduled fuel 1 st`; no premises. For `n > 1`, `run_scheduled_threaded_refines_canonical_of_round` (`Runtime/Proofs/ConcurrencyThreaded.lean:69-80`) concludes `runScheduledThreaded fuel n st = runScheduled fuel 1 st` from premise `ThreadedRoundRefinementPremises n` (`:62-65`, verbatim):
     ```
     structure ThreadedRoundRefinementPremises (n : Nat) where
       round_refines_canonical :
         ∀ st : ProtocolMachineState ι γ π ε ν,
           schedRoundThreaded n st = schedRound 1 st
     ```
   - Plain language: with one worker, the Lean threaded runner equals the canonical runner unconditionally. With `n > 1`, the theorem lifts an assumed per-round equality (for all states) to whole runs by induction on fuel; the per-round equality is the premise, not proved here. The parity doc (S5) records the `n > 1` row as "envelope-bounded parity | `ThreadedRoundRefinementPremises` (premise-scoped) | `parity_fixtures_v2.rs::envelope_bounded_parity_holds_for_n_gt_1` | Aligned under envelope contract".

5. Distributed families with placeholder status (S6). README legend (`lean/Distributed/README.md:18-23`, verbatim): "`profile shell` means the runtime profile mostly transports theorem-shaped evidence. `placeholder` means some exported guarantee is vacuous or backed by `True`-style assumptions." Placeholder rows: Nakamoto (`:34`), Reconfiguration (`:35`), FailureDetectors (`:38`), Coordination / CALM (`:40`). Profile shell: FLP, CAP, PartialSynchrony, Responsiveness, AtomicBroadcast, AccountableSafety, DataAvailability, CRDT, ConsensusEnvelope. `:48`: "No distributed family is currently marked `complete`."
   - FailureDetectors example: `impossibility_boundary_of_assumptions` (`Distributed/Families/FailureDetectors/Core.lean:173-179`) has body `a.unsupportedCompatibleDetectorCannotSolve detector`, where the `Assumptions` structure (`:76-85`) carries field `unsupportedCompatibleDetectorCannotSolve : ∀ d, DetectorClassTooWeak M d`. The theorem returns an assumption field. `Model` fields such as `solvesConsensus` and `networkTimingFaultMapping` are abstract `Prop`s (`:31-36`). Runtime checks for these assumptions report `passed := proofCarriedNoRuntimeCheck` with `def proofCarriedNoRuntimeCheck : Bool := true` (`:112-113`, `:124`, `:129`).
   - Nakamoto example: `common_prefix_of_assumptions` (`Distributed/Families/Nakamoto/Core.lean:208`) concludes `CommonPrefixAtDepth M run a.settlementDepth` by destructuring the assumption field `realizedByLongestChain` (`:113-118`: every admissible run equals the public chain of a `LongestChain.IsRun` execution whose schedule satisfies `WindowHonestMajorityUpTo` at the settlement depth) and then applying a proved `LongestChain.common_prefix`. The file comment (`:15-23`) says earlier versions "carried `chainGrowth`, `chainQuality`, and `churnWithin` premises equal to the liveness conclusions" and that quality/churn "are not modelled". The probabilistic part is an assumption: `def ProbabilityBudget ... := M.adversarialPowerBounded ∧ M.failureProbabilityAtDepth k ≤ ε` (`:67-70`), with `failureProbabilityAtDepth : Nat → Rat` an uninterpreted model field.
   - Plain language: in placeholder and profile-shell families, some exported theorems restate or transport supplied assumptions; the README says so itself. Nakamoto common prefix is derived from a deterministic longest-chain model plus a refinement assumption, not from a probabilistic adversary model.

What is guaranteed, in sum (static reading): Lean-internal theorems about a Lean protocol-machine model and session calculus. What is not guaranteed by a theorem: that the Rust runtime's steps equal the Lean steps (tested on a corpus with two trusted normalisation rules), that the compiler/macro produces the projected types (outside the claim), fairness, or the probabilistic/consensus content of placeholder families.

## 4. Property classes covered
| Class | Status | Basis |
|---|---|---|
| Functional correctness | partial | Lean model theorems (projection, step specs); Rust side tested only |
| Safety invariants | proved (Lean model) | `Coherent` preservation under `SendSpec`/`RecvSpec` (§3a.2); deadlock-freedom files listed in `CODE_MAP.md` (not read) |
| Liveness / progress | partial | Progress and scheduling-bound files exist (CODE_MAP); fairness explicitly not claimed (`schedule_nonblocking` doc); search fairness "premise-scoped" (S2) |
| Refinement | partial | Lean threaded→canonical for n=1 proved; n>1 premise-scoped; Rust↔Lean by differential tests |
| Symbolic security | no | |
| Computational security | no | SHA-256-style assumptions out of scope (S2); Nakamoto probability budget is an uninterpreted assumption |
| Parsing | no | Rust parser/lowering outside claim (S2) |
| Constant time | no | |

## 5. Link to implementation code
Mechanisms (all present):
- differential-testing (primary): Rust protocol machine vs Lean-built `protocol_machine_runner` on fixture corpora, step-indexed, through `rust/bridge` (S2, S5, S7).
- lean-is-impl: the Lean reference runner is itself an executable compiled from the Lean model; it is the oracle side of the comparison, not the shipped runtime.
Inventory, "Out of Scope" (S2, verbatim): "a blanket claim that every shipped Rust code path is mechanically proved" is not included. "shipped first-party crate artifacts are covered only by operational artifact correspondence, not by mechanized proof". Compiler boundary: `parse_choreography_str`, `choreography_to_global`, `local_to_local_r` and `tell!` expansion are "outside the formal claim"; `par`, `case/of` and `timeout` are outside even the future proof subset (S2, "Compiler and Macro Claim Boundary"). Parity doc (S5): "The bridge does not define protocol-machine semantics." and "Executable modules must not depend on placeholder proof definitions" (`docs/802_rust_lean_parity.md:496`).

## 6. Trusted computing base and soundness hygiene
Lexical counts over `lean/` (excluding `.lake`), static only:
- `sorry`: 0. `axiom` declarations: 0 (`rg "^axiom "` no hits). `native_decide`: 0. `extern`: 0. `opaque`: 0.
- `admit`: 7 word hits, all in comments/docstrings (e.g. `Protocol/SpatialBranching.lean:336` "the labels admit a separating ... code"); 0 tactic uses found.
- `@[implemented_by`: 14 attributes (`IrisExtractionInstance/Core.lean` 9, `IrisExtractionInstance/GhostAndProgramLogic.lean` 4, `IrisExtractionAPI.lean` 1). Example `Core.lean:228-235`: `fupd` "Logical: upstream Iris fancy update ... Runtime: Identity (mask tracking is proof-only)". These swap proof-only Iris constructs for runtime erasures in compiled code; they do not affect kernel-checked theorems but do affect what the compiled Lean runner executes.
- `partial def`: 1 (`Choreography/Projection/Json.lean:310 private partial def payloadsOf`).
- Classical analysis: entered as typeclass fields, not axioms. `ClassicalAnalysisAPI/Core.lean:208-213` `class Laws extends Model` with fields `shannon_entropy_nonneg`, `kl_divergence_nonneg`, etc.; a concrete `noncomputable instance realLaws : Laws` exists at `ClassicalAnalysisInstance/Models.lean:87` (instance proof bodies not read). The inventory still lists "Classical real-analysis API | external analytic laws used by transported classical families" as TCB (S2).
- Public TCB ledger (S2): Lean kernel and imported libraries; Lean model definitions; classical real-analysis API; Rust/Lean bridge normalisation and interchange; Rust runtime ("comparison-checked, not fully proved"); first-party handlers/transports; release/package scripts; Cargo/crates.io/git/toolchains.
- Heap digest "remains an operational conformance surface checked through published vectors and runtime tests rather than a first-class Lean hash model" (S2).
- Axiom inventory vs older tables: `CODE_MAP.md:1024` "`rg -n "^axiom " -g'*.lean'` returns no results. The repository is currently axiom-free." and `:1035` "| **Total** | **0** | **Fully proved** |". Older rows in the same file still read "Monitor/Preservation.lean | 187 | `MonStep_preserves_WTMon`, `newSession_preserves_WTMon` — 2 axioms" (`:560`) and "Deployment/Linking.lean | 87 | Composition metatheory — 8 axioms" (`:568`). At this commit `Protocol/Monitor/Preservation.lean` is 451 lines with 0 `^axiom` and `Protocol/Deployment/Linking.lean` is a 7-line barrel importing `LinkingCore` and `LinkingTheorems`, 0 `^axiom`. The per-file rows are stale relative to the code; the global statement matches the lexical check.

## 7. Scale, effort, automation
- Lean: 164,896 lines, 830 files, 5,105 theorem/lemma declarations (lexical; CODE_MAP's own scope: 767 files / 152,326 lines). Rust: 234,015 lines.
- Dependencies include Mathlib and iris-lean.
- Single author. Effort not reported. Automation: CI workflows `verify.yml`, `check.yml`, `release-preflight.yml`, `supply-chain.yml` (names only; not read); gates `just verify-protocols`, `just check-bridge-normalization`, `just artifact-check`.

## 8. Limitations stated by the authors (verbatim)
- "This does not mean every Rust API, compiler helper, macro path, plugin, or third-party integration is mechanically proved." (S2)
- "| Rust runtime implementation | shipped executable semantics are still comparison-checked, not fully proved | ..." (S2, Public TCB Ledger)
- Refinement map: "Effect exchanges and output-condition trace" are "outside the current claim-critical refinement core"; "Semantic-object families" are "theorem-backed separately, outside the current claim-critical refinement core" (S2).
- Simulator lanes "remain executable assurance lanes rather than part of the current mechanized formal claim" (S2).
- Search fairness theorems are "premise-scoped" and "still narrower than a blanket proof of fairness and completeness" (S2).
- "No distributed family is currently marked `complete`. ... many profiles still carry theorem-shaped evidence or depend on vacuous placeholders." (S6, `lean/Distributed/README.md:48-50`)
- Parity doc deviation registry lists active runtime-only deviations: rust-host-targeted-disposal, rust-host-worker-pool-error-source, rust-host-owned-session-parity, rust-communication-hash-model (S5, "Deviation Governance").

## 9. Independent evidence
None found. No third-party audit, citation or reproduction located; papers not peer-review-status-checked (§15).

## 10. Reproducibility
Not built: sandbox disk limit; queued for CI reproducibility job. Exact commands: `git clone https://github.com/hxrts/telltale && cd telltale && git checkout c8c6a7fb3700896d1ab31f4bb00e42de97682943 && cd lean && lake exe cache get && lake build`; then from repo root `just verify-protocols`, `just check-bridge-normalization`, `cargo test -p telltale-bridge --test protocol_machine_differential_steps` (strict mode needs `lean/.lake/build/bin/protocol_machine_runner`), `just artifact-check`.

## 11. Generalisation
Pattern: a Lean model whose executable form serves as an oracle for a separately written Rust runtime, with correspondence established by step-indexed differential tests over a fixture corpus and a documented, narrow normalisation layer. The Lean theorem defines the compared surface. This pattern applies to any language that can call a Lean executable over JSON; it gives test-level, not proof-level, assurance for the implementation.

## 12. Bearing on sub-claims (facts only, no verdict)
- H1: Lean expresses asynchronous buffered MPST, a coroutine scheduler, threaded execution and abstract distributed-system models (§3, §3a). Probability enters only as uninterpreted fields (`failureProbabilityAtDepth : Nat → Rat`) in the Nakamoto family.
- H2: 5,105 lexical theorem declarations with 0 sorry/axiom; coherence preservation proved under step specs; several distributed families export theorems that return assumption fields (§3a.5), as the author labels.
- H3 / H5b: The Rust runtime is linked to the Lean model by differential testing against a Lean-built runner, not by a proof; the author states this ("comparison-checked, not fully proved"). Bears on H5b: implementation checking without a Rust-side proof.
- H4: Mechanism is language-agnostic at the JSON boundary; only one implementation language (Rust) is exercised.
- D: The project uses "formally verified" for a declared surface that includes operationally checked artefacts and comparison-checked runtime code; "verified" here covers proof, differential testing and artefact provenance checks under one claim (S1, S2).

## 13. Conflicts
- CODE_MAP per-file rows say Monitor/Preservation has "2 axioms" and Deployment/Linking "8 axioms" (`CODE_MAP.md:560,568`), while the same file's global inventory (`:1024`, `:1035`) and a lexical check of the current files find 0 axioms. File sizes in those rows (187, 87 lines) also differ from the current files (451, 7 lines).
- `lean/Distributed/README.md:34` labels Nakamoto "placeholder" while `Nakamoto/Core.lean:208` proves common prefix from a longest-chain refinement assumption plus `LongestChain.common_prefix`; the probability budget remains an assumption. Label and code differ in degree.
- Line/file totals: CODE_MAP and inventory 767 files / 152,326 lines vs my lexical 830 / 164,896 (different scope). research/01-lean4-protocol-modeling.md S15 recorded 5,101 theorems, implemented_by 8, partial 1; my counts are 5,105, 14 attributes, 1 (different counting rules and/or commit).

## 14. Sources
| ID | Title | URL | P/S | Depth | Summary |
|---|---|---|---|---|---|
| S1 | hxrts/telltale README @ c8c6a7f, accessed 2026-11-07 | https://github.com/hxrts/telltale/blob/c8c6a7fb3700896d1ab31f4bb00e42de97682943/README.md | P | [FULL] | Project overview, three components, public verification claim and exclusions. |
| S2 | `docs/902_verification_inventory.md` @ c8c6a7f (483 lines), accessed 2026-11-07 | https://github.com/hxrts/telltale/blob/c8c6a7fb3700896d1ab31f4bb00e42de97682943/docs/902_verification_inventory.md | P | [FULL] | Claimed surface, out-of-scope list, final surface audit, public TCB ledger, refinement coverage map, bridge normalisation trust surface. |
| S3 | `lean/Runtime/Proofs/ProtocolMachine/ConcreteRefinement.lean` @ c8c6a7f (430 lines), accessed 2026-11-07 | https://github.com/hxrts/telltale/blob/c8c6a7fb3700896d1ab31f4bb00e42de97682943/lean/Runtime/Proofs/ProtocolMachine/ConcreteRefinement.lean | P | [FULL] | Concrete refinement slice definitions and the claimed-surface, coherence, determinism and threaded theorems. |
| S4 | Lean source tree `lean/` @ c8c6a7f (SchedulerContracts, EdgeCoherenceCore, InstrSpec, ConcurrencyThreaded, ClassicalAnalysisAPI, IrisExtractionInstance, Monitor/Preservation, Deployment/Linking), accessed 2026-11-07 | https://github.com/hxrts/telltale/tree/c8c6a7fb3700896d1ab31f4bb00e42de97682943/lean | P | [CODE] | Predicate definitions quoted in §3a; lexical hygiene counts in §6. |
| S5 | `docs/802_rust_lean_parity.md` @ c8c6a7f (1,081 lines), accessed 2026-11-07 | https://github.com/hxrts/telltale/blob/c8c6a7fb3700896d1ab31f4bb00e42de97682943/docs/802_rust_lean_parity.md | P | [SECT] | Sections read: Contract Levels, Bridge Scope, Lean Runners, Threaded Concurrency Envelope, Deviation Governance. Quote: "Parity is enforced at two levels. Level 1 covers policy and data shape parity ... Level 2 covers behavior parity for executable traces under the declared concurrency envelope." |
| S6 | `lean/Distributed/README.md` plus `Families/FailureDetectors/Core.lean` and `Families/Nakamoto/Core.lean` @ c8c6a7f, accessed 2026-11-07 | https://github.com/hxrts/telltale/blob/c8c6a7fb3700896d1ab31f4bb00e42de97682943/lean/Distributed/README.md | P | [SECT] | Sections read: legend, family status table, "Current Completion Boundary"; two family Core files read at cited lines. Quote: "No distributed family is currently marked `complete`." |
| S7 | `rust/bridge/tests/protocol_machine_differential_steps.rs` @ c8c6a7f (558 lines), accessed 2026-11-07 | https://github.com/hxrts/telltale/blob/c8c6a7fb3700896d1ab31f4bb00e42de97682943/rust/bridge/tests/protocol_machine_differential_steps.rs | P | [CODE] | Function list read (runner setup, canonicalisation, `assert_step_indexed_equivalence`, tier1/tier2 tests); bodies not read in full. |
| S8 | `lean/CODE_MAP.md` @ c8c6a7f (1,132 lines), accessed 2026-11-07 | https://github.com/hxrts/telltale/blob/c8c6a7fb3700896d1ab31f4bb00e42de97682943/lean/CODE_MAP.md | P | [SECT] | Sections read: totals header, Runtime Monitoring, Deployment Composition, Axiom Inventory. Quote: "The repository is currently axiom-free." |

## 15. Unverified leads (excluded from evidence)
- Papers 1-3 (https://hxrts.com/telltale/papers/paper1.pdf, paper2.pdf, paper3.pdf; LaTeX in `papers/`): only titles and the paper 1 abstract were read. Titles: "Coherence for Asynchronous Buffered MPST: A Mechanized Metatheory"; "Computable Dynamics for Asynchronous MPST: Lyapunov Descent, Critical Capacity, and Decision Procedures"; "Harmony from Coherence in Asynchronous MPST: A Minimal Erasure Invariant for Classical Dynamics". Venue/peer-review status unknown.
- Whether `execInstr` is proved to satisfy `SendSpec`/`RecvSpec` for each instruction (not traced).
- `ClassicalAnalysisInstance/Models.lean` instance proof bodies (not read).
- Rust test `parity_fixtures_v2.rs::envelope_bounded_parity_holds_for_n_gt_1` and `lean_trace_validation.rs` (names only).
- CI workflow contents and whether CI passes at this commit (not checked).
