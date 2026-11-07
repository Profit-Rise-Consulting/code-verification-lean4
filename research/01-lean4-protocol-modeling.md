# 01 - Lean 4 (and Lean 3 lineage) protocol modeling: evidence file (Phase 1b)

Counts (this file): [FULL] entries: 26 (S7, S10, S12-S14, S17-S23, S30-S32, S34, S36-S38, S40, S42 paper, S45-S48 plus S35 slides/blog) | [CODE] entries: 29 (6 of them, S10, S19, S21, S22, S31, S36, are also FULL, listed as FULL+CODE; S24 and S35 are counts-only CODE) | Unverified leads (excluded from evidence): 18 | Access date for every URL: 2026-11-07.
Counting rule: an entry is FULL if the full text of the document was retrieved (PDF text, HTML page or raw file) and the quoted passages were located and read in that text (documents were searched by keyword, not read linearly, when they exceed about 30 pages); CODE if the repository was cloned (`--depth 1`, blob limit 1 MB) or its files were downloaded and inspected. FULL+CODE entries are counted in both totals. Phase 1 entries that were search-snippet-only have been removed from the evidence and listed under "Unverified leads".

Neutrality: this file records what sources claim, what they demonstrate, and what they say they do not cover. It contains no verdict on the hypothesis. Sub-claim tags (H1-H5, D) follow `research/CONVENTIONS.md`.

Scope pointers: deep per-project dossiers live in `research/references/` (Cedar, SampCert, LNSym, Veil/Lentil/Velvet/Loom, Aeneas/SymCrypt, hax, VCVio, tls13-lean, Clean/zkEVM, LeanMqtt, Std.Http, libp2p-fv, raft-lean-squad, leanda, leslie, verified-consensus, Everest, IronFleet, Verdi, Tamarin/ProVerif, Ivy QUIC, TLA+ trace validation). This file gives only the modeling-landscape view and the facts that this agent verified itself; for those projects it records the repo metrics and a short factual summary and points to the dossier for depth.

## 0. Method notes and measurement caveats

- Repo metrics come from a shallow clone in `/tmp`. Counts are produced by a small script that strips Lean comments (`--`, nested `/- -/`) and string literals, then counts whole-word tokens: `sorry`, `admit`, `axiom` (declaration at line start), `native_decide`, `implemented_by`, `@[extern ...]`, `opaque` (declaration), `partial def`, and `theorem`/`lemma` declarations. Counts can include tokens inside metaprogramming code (e.g. code that emits `sorry`), so they are upper-bound indicators, not audits. `#print axioms` was not run (no Lean toolchain or builds, per disk limit instruction).
- A naive `grep -w sorry` gave higher numbers than the comment-stripped count for several repos (for example raft-lean-squad 29 vs 0; Leslie 41 vs 5), because prose and comments mention the word. The stripped counts are used below.
- LOC = all lines in `*.lean` files excluding `.lake`; "code" = non-blank, non-comment lines.
- Projects whose clone was removed to free disk before every field was recorded have a smaller field set (stated in the entry).
- AI-generated flag: rule 7 of CONVENTIONS requires this. Flags below state the basis (README statement, file names such as `AGENTS.md`/`CLAUDE.md`, or commit-pattern evidence). Commit-pattern analysis was not possible with `--depth 1` clones, so flags rest on README/file evidence only.
- The GitHub REST API was used only for: the Lean 3 repo list in the Zulip thread, commit SHA of `leanprover/lean4` master, and a file tree listing for `src/Std/Http`. No metrics were taken from it.

## 1. Source register

### 1A. State machines, transition systems, refinement (Event-B lineage)

**S1. LeanMachines (library) [P][CODE] tags: H1, H2, D**
https://github.com/lean-machines-central/lean-machines | SHA 9f1f001f0a | last commit 2026-11-04 | toolchain `leanprover/lean4:v4.35.0-rc3` | licence present (LICENSE) | 50 files, 11,252 LOC (7,992 code) | sorry 10 by the stripped count (comment-stripped; plain grep finds 16 tokens in `NonDet/Algebra/Convergent.lean`, `NonDet/Algebra/Ordinary.lean` and `Algebra/Arrow.lean`) | axiom 4 by the script, 5 by plain grep: `_Action_ext_ax` (`Event/Basic.lean:106`), `AnticipatedEvent.ext` (`Event/Convergent.lean:96`), `ConvergentEvent.ext` (`Event/Convergent.lean:319`), `_Effect_ext_ax` (`NonDet/Basic.lean:42`), `OrdinaryNDEvent_split_feasibility_ax` (`NonDet/Algebra/Ordinary.lean:276`) | theorem 23 | native_decide 0.
- README: "dedicated to the formal modeling of stateful systems. The main objective is to support a stepwise refinement methodology inspired by the Event-B formal method"; "the framework enforces the fundamental principle of correctness-by-construction: machine states, events structures and refinement steps cannot be fully constructed without discharging the prescribed proof obligations." Status line: "Project status : alpha ... may be modified without prior warning." Also: "The framework is not directly compatible with Event-B and related implementations such as Rodin."
- Core definition (`LeanMachines/Event/Basic.lean`): `class Machine (CTX) (M) extends Inhabited M where context : CTX; invariant : M -> Prop`, doc: "an `invariant` property defining the overall safety requirements of the machine states". Events are `structure _Event ... where guard (m : M) (x : alpha) : Prop := True; action (m : M) (x : alpha) (grd : guard m x) : (beta x M)`. Event kinds: `InitDet | InitNonDet | TransDet (status : Convergence) | TransNonDet ...` with `Convergence = Ordinary | Anticipated | Convergent`.
- Axioms (see list above): `_Action_ext_ax` in `Event/Basic.lean` carries the author comment "XXX : does this axiom breaks something ? (I don't think it's provable because of HEq)"; the others are extensionality or feasibility axioms for events (listed above).
- Refinement modules: `Refinement/{Functional,Relational,Bidirectional}`; `FRefinement`, `BiRefinement`, `Convergent` refinement specs with a variant `v` and `WellFoundedLT v`. A comment in `Refinement/Relational/Concrete.lean` says a point "is more related to a fairness argument". No LTL/temporal-logic module, no fairness definition and no network/message model in the library (grep for liveness/temporal/deadlock/fairness returned only that comment).
- What it demonstrates: a typeclass-and-structure encoding of Event-B machines with proof-obligation-carrying refinement, safety as state invariants, termination-style convergence via variants. What it does not state: message passing, adversary, time, probability, links to code.

**S2. lean-machines-examples [P][CODE] tags: H1, H2, H3**
https://github.com/lean-machines-central/lean-machines-examples | SHA 4544600e38 | 2026-11-06 | toolchain v4.35.0-rc3 | licence present | 31 files, 9,637 LOC | sorry 25 (EventB/Courses/M2.lean 6, MQueue/MQ3.lean 6, Algebraic_examples/Bounded.lean 3, others), axiom 2, theorem 227.
- Example directories: `Algebraic_examples`, `Buffer`, `EventB`, `MQueue` (MQ0 to MQ4, Bounded, Clocked, Prioritized), `VDM`. The message-queue examples are single-component queue refinements; none is a multi-node network protocol. No code-extraction or link to another language is present in the repo files inspected.

**S3. LeanMachines papers (IFM 2024 / ACM) [P][not read]**
Listed under Unverified leads (ACM and HAL pages returned no readable text from this environment).

### 1B. TLA-style encodings, temporal logic, safety vs liveness

**S4. Leslie (TLA shallow embedding, refinement, round-based protocols) [P][CODE] tags: H1, H2, D**
https://github.com/rupakm/leslie | SHA 58e58b2502 | last commit 2026-06-09 | toolchain `leanprover/lean4:v4.27.0` | LICENSE present | 167 files, 103,805 LOC (77,303 code) | theorem 2,216 | sorry 5 (Leslie/Refinement.lean 2, Leslie/Examples/LastVoting.lean 2, Leslie/Prob/Spike/ASTSanity.lean 1) | native_decide 1 | partial 14 | axiom 0.
- Core definitions (`Leslie/Refinement.lean`): `structure Spec (sigma) where init : sigma -> Prop; next : action sigma; fair : List (action sigma) := []`, with `Spec.safety`, `Spec.formula`, and refinement lemmas `refinement_mapping_nostutter`, `refinement_mapping_stutter`, `refinement_mapping_with_invariant`. The two `sorry` in this file (lines 387 and 515) sit at the backward-simulation part (comment at line 513: "For now, we sorry this and note that the backward simulation itself is...").
- Example inventory (file names): `Examples/Paxos/{PaxosRefinement, MessagePaxos, MsgPaxosConsensus, BoundedPaxos, BoundedSingleProposer}`, `LastVoting`, `BenOr`, `BallotLeaderPhased`, `PetersonLiveness` (leads-to theorems such as `waiting1_true_leads_to_cs1`), `BallotLeaderLiveness`, `SnapshotRefinement`, `Combinators/PhaseCombinator`, and `Examples/Prob/{BrachaRBC, AVSS, AVSSAbstract, AVSSFaithful, CommonCoin, BenOrAsync, Shamir, ...}`. Probabilistic files discuss "FairASTCertificate" and almost-sure termination under fairness assumptions (for example `BrachaRBC.lean` section 8, `brbProb_totality_AS_fair`, "liveness; **closed** sorry-free").
- Self-reports about `sorry` disagree across files (see C1): README table says "Paxos | Quorum intersection, 14 actions | Spec complete, 2 sorry" and "VRViewChange | ... | 1 sorry"; `Examples/Paxos/PaxosRefinement.lean` docstring says "Current design (abstract-ahead, 1 sorry)" and "Remaining sorry: The p2a max-vote constraint (`h_value_ok`)"; `Examples/Paxos/PaxosEvaluation.md` states "All proofs are machine-checked and sorry-free. The development totals 4584 lines of Lean 4 across four files." The comment-stripped scan finds no `sorry` token in the Paxos files; the 5 tokens are in `Refinement.lean` (2), `Examples/LastVoting.lean` (lines 304 and 399; docstring "We leave this as sorry"), `Prob/Spike/ASTSanity.lean` (1). Prose in `BenOr.lean` (line 832) and `BallotLeaderPhased.lean` (line 260) also says "sorry" is used, but the scan counts no token there.
- Rust link: `docs/rust-verification.md` (593 lines) states the goal: "Given a round-based protocol proved safe in Leslie and a Rust implementation using the `Communication`/`InboxExt` API, establish that the Rust code correctly implements the Leslie model - and therefore inherits the Leslie safety proof." It sketches, inside a documentation code block, a theorem `rust_correctness` (hypotheses `encode`, `h_init`, `h_send`, `h_update`, `h_closure`; the conclusion is written as `...  -- the Rust system satisfies the Leslie safety property`), whose per-protocol hypotheses are to be discharged "in Verus on the Rust side". The same document lists assumptions: "Transport correctness assumed ... We do not verify the transport" and "We defer the question of how to discharge them (Verus solver, manual audit, or testing)." `docs/rust-verification-plan.md` is headed "Status: UNDER REVIEW". The README also references `rust-verification-v2.md` and lists `Leslie/Rust/CoreSemantics.lean` and `RuntimeSemantics.lean`; the cloned tree has no `Leslie/Rust` directory (C2). The `rust/` crate has `comm.rs`, `driver.rs`, `protocol.rs`, `ballot_leader_phased.rs` and tests `one_third_rule.rs`, `lease_lock.rs`.
- AI-generated flag: repo root contains `AGENTS.md` and `CLAUDE.md` (the latter starts with SageOx agent-priming instructions) and many `PHASE-*-CHECKPOINT.md` files; basis is file presence. See dossier.

**S5. lean-tla (TLA+ DSL in Lean, with vendored Leslie) [P][CODE] tags: H1, H2, D**
https://github.com/yihuang/lean-tla | SHA 99d8be4103 | last commit 2026-08-17 | toolchain `leanprover/lean4:v4.33.0-rc2` | no LICENSE file found at root | 323 files, 141,774 LOC (104,397 code) | theorem 3,404 | sorry 5 (all in `docs/research/vendor/leslie`, the vendored Leslie copy; none in the repository's own `TlaDsl`/`Bft` code) | native_decide 7 | partial 50.
- README: "Exploration workspace for a **deep embedding of TLA+ in Lean 4**, with Isabelle/TLA+ as the reference and mathlib/CSLib as leverage." Docs: `design-space.md`, `deep-vs-dsl.md`, `implementation-strategies.md`, `north-star.md` (goal: formalize McMillan's relational rankings, CAV 2024, "Toward Liveness Proofs at Scale", as the liveness engine; "M1, the basic rule + queue example, is done"), `ux-notes.md`. Code: `TlaDsl/` (typed DSL with `Tla.Entails`, `Tla.leadsTo`, `Tla.statePred`) and `Bft/`, examples including `Paxos.lean`, `StreamletByzLiveness.lean`. Root has `AGENTS.md`; no LICENSE file. AI-assistance flag rests on `AGENTS.md` and the volume of `docs/`; no explicit statement found.
- Verbatim liveness statement (`TlaDsl/Examples/StreamletByzLiveness.lean:1293`):
  ```lean
  theorem liveness_spec_byz (n : Nat) (Byz : Finset Node) (L : Epoch -> Node)
      (hByz : ByzOk n Byz) (e0 : Epoch) (he0 : 0 < e0) :
      Tla.Entails (HByz n Byz L)
        (Tla.leadsTo (Tla.statePred (fun s => s.cur = e0))
          (Tla.statePred (fun s => FinalSomeByz n s))) := by
  ```
  Docstring: "with `f < n/3` Byzantine nodes, under the clock, the honest leaders proposing, and the honest nodes voting, from the start of a 5-epoch window some block is eventually final - for any leader schedule `L` whose leaders are honest." The leads-to is under the spec `HByz` (parameterised by schedule `L` and fault set `Byz` with `ByzOk n Byz`); the clock, honest-leader and honest-voting conditions are hypotheses of the theorem.
- `Paxos.lean` header: "We port Lamport's abstract Voting spec (`PaxosHowToWinATuringAward/Voting.tla`) rather than the message-passing algorithm itself", and proves `ShowsSafety`, `next_inv`, `spec_entails_agree`.

**S6. Lentil (LTL proof mode for Lean 4) [P][CODE] tags: H1, H2**
https://github.com/verse-lab/Lentil | SHA 1ddf1fc161 | last commit 2026-11-06 | toolchain `leanprover/lean4:v4.35.0` | LICENSE present | 83 files, 12,004 LOC (7,744 code) | theorem 318 | sorry 0 | axiom 0 | opaque 2 | partial 16.
- README: "a proof infrastructure for linear temporal logic in Lean 4. It formalizes the semantics of LTL, provides rules and a dedicated proof mode to conduct proofs in LTL as an object logic." Acknowledgments: definitions "initially ported from `coq-tla`"; proof mode "partly inspired by Iris Proof Mode".
- `Lentil/Basic.lean` verbatim: `def always (p : pred a) : pred a := fun e i => forall k, p e (i + k)` (line 65); `def Lentil.leads_to p q := [tlfml| box (p -> diamond q)]` (199); `Lentil.weak_fairness a := box ((box (Enabled a)) -> diamond <a>)` (201); `Lentil.strong_fairness a := box ((box diamond (Enabled a)) -> diamond <a>)` (202). `LentilApplications/RelationalRanking` is the application directory. Dossier: `references/Veil-Lentil-Velvet-Loom`.

**S7. LeanLTL paper (ITP 2025) [P][FULL] tags: H1**
https://arxiv.org/pdf/2507.01780 (also https://drops.dagstuhl.de/entities/document/10.4230/LIPIcs.ITP.2025.37 retrieved; text read).
Abstract: "We propose LeanLTL, a unifying framework for linear temporal logics in Lean 4. LeanLTL supports reasoning about traces that represent either infinite or finite linear time. ... We prove that standard flavors of LTL can be embedded in our framework." Future work (verbatim): "we aim to integrate LTL and LTLMT decision procedures directly into our library"; "We also plan to consider adding support for other LTL variants, including past-time and bounded-time operators." Examples are cyber-physical/system traces, not network protocols (no fairness/leads-to section found in the paper text).

**S8. LeanLTL repository [P][CODE] tags: H1, H2**
https://github.com/UCSCFormalMethods/LeanLTL | SHA 56ca11b3d6 | 2026-10-20 | toolchain `leanprover/lean4:v4.17.0-rc1` | LICENSE present | 32 files, 4,408 LOC (3,141 code) | theorem 309 | sorry 14 (all in `LeanLTL/Logics/LTLfMT.lean`) | axiom 2 (`LeanLTLExamples/TeaserITP2025.lean`) | partial 3.

**S9. LeanearTemporalLogic (student LTL project) [P][CODE] tags: H1**
https://github.com/mrigankpawagi/LeanearTemporalLogic | SHA 24796b51e0 | 2025-07-27 | toolchain v4.18.0-rc1 | no licence file | 11 files, 4,244 LOC | theorem 70 | sorry 15 (all in `Satisfaction/TransitionSystems.lean`). README states it is a project on LTL and transition systems and lists prior Lean 3 temporal-logic repositories (see S40-S43). Incomplete (15 sorry).
- README read: "a formalization of Linear Temporal Logic (LTL) in the Lean 4 theorem prover"; "work in progress and may not be ready for use as a dependency"; it states "I am aware of at least four Lean 3 projects aiming to formalize LTL ... Since none of these are maintained or have been ported to Lean 4", listing unitb/temporal-logic, loganrjmurphy/lean-temporal, James-Oswald/linear-temporal-logic and GaloisInc/lean-protocol-support (`galois/temporal`).

**S10. Zero-to-QED, chapter 22 "State Machine Verification" and repo [S][FULL+CODE] tags: H3, D**
https://sdiehl.github.io/zero-to-qed/22_model_checking.html and https://github.com/sdiehl/zero-to-qed | SHA cf59ccfb80 | 2026-11-01 | toolchain v4.34.1 | LICENSE present | 50 files, 9,045 LOC | theorem 405 | sorry 3 (tutorial files `Proving.lean`, `AlgebraicStructures.lean`, `Tactics.lean`) | axiom 2 (`Basics.lean`) | native_decide 7 | partial 5. Tutorial book (secondary).
- Chapter (re-read in full at access date) opens: "real systems are not written in Lean. They are written in Rust, C, Go, or whatever language the team k[nows]"; it covers verification-guided development, model-based testing and bounded model checking. Circuit-breaker state machine in Lean; Rust `step` function compared with it: "The test checks exact output equality for the 83,300 generated single-step cases: thresholds 1-4, timeouts 1-10, and the enumerated states and events" and a trace test of "2,025 executions over thresholds 1 to 3, timeouts 1 to 3, and timestamps up to 5". Verbatim limitations: "Transferring a theorem requires semantic correspondence: testing supports confidence in that correspondence but does not generally prove it."; "A proof about the model applies to the implementation only where this correspondence holds."; "These tests provide evidence of correspondence, not a proof for all inputs."; "A finite abstraction can sometimes support an unbounded theorem, but its completeness and its connection to the implementation must themselves be justified." Rust side uses `saturating_sub`/`saturating_add(1)` to match natural-number subtraction/overflow behaviour. The chapter attributes the approach to AWS and Cedar.
- Repo `sdiehl/zero-to-qed` includes `src/ZeroToQED/CircuitBreaker.lean`. Secondary (tutorial); used for how a Lean-model-to-Rust correspondence is presented, not as an authoritative result.

### 1C. Process calculi, labelled transition systems, impossibility results

**S11. CSLib repository [P][CODE] tags: H1, H2**
https://github.com/leanprover/cslib | SHA 76b7943d0b | 2026-11-05 | toolchain `leanprover/lean4:v4.36.0-rc2` | LICENSE present | 341 files, 52,113 LOC (30,796 code) | theorem 3,050 | sorry 0 | axiom 0 | native_decide 0 | opaque 2.
- Directory inventory relevant to protocols: `Cslib/Foundations/Semantics` (LTS), `Cslib/Languages/CCS`, `Cslib/Languages/Mech` (including a choreography area), `Cslib/Logics/HML`, `Cslib/Logics/Modal`, `Cslib/Computability/Distributed` (subfolders `FLP` and `Consistency`), `Cslib/Crypto/Protocols`, `Cslib/Algorithms/StatefulProcesses`. A search of the tree for "raft" returned no Raft development (only unrelated word matches).
- `Cslib/Computability/Distributed/FLP/README.md` (author Ching-Tsun Chou, "Copyright (c) 2026", Apache 2.0): "This directory contains a formalization of Volzer's proof [Volzer2004] of the famous result in distributed computing, first proved by Fischer, Lynch and Paterson [FLP1985], that asynchronous distributed consensus is impossible in the presence of even a single crash fault." Eight files (`Algorithm`, `Consensus`, `FairScheduler`, `CanReachVia`, `PseudoConsensus`, `OnePseudoConsensus`, `Impossibility`, `ZeroConsensus`). `ZeroConsensus.lean` is "included to show that the notion of an algorithm ... is not vacuous". Final theorems in `Impossibility.lean`: `OnePseudoConsensus.not_terminating`, `Consensus.one_not_exists [Fintype P] (hc : 2 <= card P)`, `Consensus.ge_one_not_exists ... (hf : f >= 1)`. This is a proof about the class of all algorithms in the model, not about one protocol instance.
- `Computability/Distributed/Consistency/Specification.lean` formalizes "several notions of consistency for replicated data types defined in [Burckhardt2014]" ordered by strength.
- Dossier: none specific; see also S12-S14.
- FLP verbatim (read from source): `FLP/Algorithm.lean` models an algorithm as a labelled transition system over process states and a message pool; `FLP/FairScheduler.lean` defines fair executions; `Impossibility.lean` ends with `theorem Consensus.one_not_exists ...` and `Consensus.ge_one_not_exists`; the README cites Volzer's proof and states the result for "even a single crash fault". The development is a proof over all algorithms satisfying the model's definitions; it does not verify a specific protocol or any code.
- `Cslib/Languages/Mech/README.md` (author Fabrizio Montesi, Apache 2.0): "This directory is a placeholder for the upstreaming of Mech, a language for choreographic programming developed at FORM." It lists plans: a "library of verified protocol for concurrent and distributed systems" and a choreography compiler with "strong bisimilarity". Only `Choreography/Basic.lean` exists at this SHA.
- `Cslib/Foundations/Semantics/LTS/` has `Basic`, `Bisimulation`, `Simulation`, `TraceEq`, `Execution`, `OmegaExecution`, `Divergence`, `Termination`, `Total`; `Cslib/Languages/CCS/` has `Basic`, `Semantics`, `BehaviouralTheory`; `Cslib/Algorithms/StatefulProcesses/DiffieHellman`. No temporal-logic (LTL) module and no network model were found in the tree.

**S12. CSLib white paper (arXiv 2602.04846) [P][FULL] tags: H1, H4**
https://arxiv.org/pdf/2602.04846. States CSLib "aims to be for computer science what Lean's Mathlib is for mathematics". Pillar 1 topic figure lists "Process calculi", "Petri nets", "Concurrency", "Choreographic languages". Pillar 2 text: future work includes "support for reasoning about concurrency, and support for reasoning about low-level constructs such as pointers", citing "choreographic programming and separation logic". Concurrency code reasoning is therefore stated as planned work in the paper.

**S13. "Computer Science as Infrastructure: the Spine of CSLib" (arXiv 2602.15078, v2 July 2026) [P][FULL] tags: H4**
https://arxiv.org/abs/2602.15078 and https://arxiv.org/html/2602.15078. Describes the library architecture. A keyword search of the retrieved text for "distributed", "concurrent" and "protocol" returned no hits in the HTML body; the paper does not discuss protocol verification.

**S14. Hennessy-Milner Logic in CSLib (arXiv 2602.15409) [P][FULL] tags: H1, H2**
https://arxiv.org/pdf/2602.15409. Abstract: "library-level formalisation of Hennessy-Milner Logic (HML) ... syntax, satisfaction relation, and denotational semantics of HML, as well as a complete metatheory including the Hennessy-Milner theorem - bisimilarity coincides with theory equivalence for image-finite LTSs." Intro lists "communication protocols" among systems LTSs describe; no protocol case study is formalized in the paper.

### 1D. Session types and choreographies

**S15. Telltale (Rust choreography library + Lean proof stack) [P][CODE] tags: H1, H2, H3, D**
https://github.com/hxrts/telltale | SHA c8c6a7fb37 | 2026-10-31 | toolchain `leanprover/lean4:v4.35.0` | LICENSE present | 830 Lean files, 164,896 LOC (114,456 code) | theorem 5,101 | sorry 0 | axiom 0 | native_decide 0 | implemented_by 8 | partial 1.
- README: "A Rust library for writing composable multiparty session-typed choreographies, an extensible Lean proof system, and a three-paper series"; Rust DSL features include "Simulation, replay, and cross-target conformance tooling against Lean reference traces". Formal claim sentence: "Telltale is formally verified for the declared shipped surface documented in docs/902_verification_inventory.md, under the listed public assumptions and trusted base. The claim covers the Lean semantics and theorems, the theorem-defined Rust<->Lean core protocol-machine runtime correspondence, the shipped first-party handler and transport contract boundary ..."
- `docs/902_verification_inventory.md` (483 lines) trusted computing base table, verbatim rows: "Rust runtime implementation | shipped executable semantics are still comparison-checked, not fully proved"; "Lean model definitions | theorems are only as correct as the modeled semantics"; "Rust/Lean bridge normalization and interchange | comparison/equality surface between Rust and Lean". Statement of scope: "the current public formal-verification claim does not include any Rust parser, lowering, projection, import/export, or macro-expansion entry path." Out of scope: "User-supplied third-party handlers, transports, plugins, and deployment adapters".
- Single-author repository with very large volume (164k Lean LOC); AI-generation status not stated in the README; not determined.
- Status legend inside `lean/Distributed/README.md` (read): entries in the distributed-family inventory are marked with statuses including `placeholder`, defined as "some exported guarantee is vacuous or backed by True-style assumptions". Rows `Nakamoto` and `Reconfiguration` carry the `placeholder` status in that table; protocol-level theorems for each family were not individually read.
- Internal inconsistency recorded in C7: `lean/CODE_MAP.md` says in its Axiom Inventory "`rg -n "^axiom " -g'*.lean'` returns no results. The repository is currently axiom-free", while the same file's file table still carries rows such as "Monitor/Preservation.lean ... 2 axioms" and "Deployment/Linking.lean ... 8 axioms".

**S16. rumpsteak-aura (Rust choreographic MPST with Lean directory) [P][CODE] tags: H1, H2, H3**
https://github.com/hxrts/rumpsteak-aura | SHA 024b769afe | last commit 2026-01-30 | LICENSE present | 221 Lean files, 58,185 LOC (41,738 code) | theorem 2,287 | axiom 35 (Effects/Deployment/Part2b.lean 9, Effects/DeadlockFreedom.lean 7, Effects/Preservation.lean 6, Effects/Deployment/Part1.lean 6, others) | native_decide 2 | partial 5 | sorry 0.
- README: "a Rust framework for safely and efficiently implementing message-passing asynchronous programs. It uses multiparty session types to statically guarantee the absence of communication errors such as deadlocks"; session-type core "forked from Zak Cutner's Rumpsteak". The Lean `Effects/DeadlockFreedom.lean` file declares 7 axioms, i.e. its deadlock-freedom statement rests on stated assumptions in Lean. Same author as S15 (hxrts); the relationship between the two repositories was not documented in the files read.
- Read from source: `lean/Effects/DeadlockFreedom.lean` states in its module doc that it "provides deadlock freedom infrastructure for MPST" and declares, as `axiom`, the headline results `deadlock_free (C : Config) (Ssh Sown : SEnv) ...` (line 319) and `not_stuck (C : Config) (Ssh Sown : SEnv) ...` (line 326), plus `reachesComm_body_implies_unfold` and `reachesCommDecide_sound`. So the deadlock-freedom statements in this repo are assumed in Lean, not proved there.

**S17. "Session Type State Spaces Form Lattices" (EPTCS 453) [P][FULL] tags: H1**
https://arxiv.org/pdf/2609.34927. Abstract-level facts from the text: "Mechanised in Lean 4 with two independently developed tool implementations"; "in Lean 4 (17 modules, zero sorry)"; classification of 93 distributive and 15 non-distributive cases. A structural result about session-type state spaces; no network or implementation link.

**S18. Asynchronous multiparty session types, mechanised in Rocq (ECOOP 2025) [P][FULL] non-Lean comparator; tags: D**
https://drops.dagstuhl.de/storage/00lipics/lipics-vol333-ecoop2025/LIPIcs.ECOOP.2025.31/LIPIcs.ECOOP.2025.31.pdf. Included only as comparator for what mechanised MPST covers outside Lean. Not used for Lean claims.

### 1E. Distributed protocol and consensus developments in Lean 4

**S19. Veil (framework for transition systems; distributed-protocol DSL) [P][FULL+CODE] tags: H1, H2, H3, D**
https://github.com/verse-lab/veil | SHA 57447722e7 | 2026-11-07 | toolchain v4.35.0 | LICENSE present | 495 files, 89,474 LOC (56,556 code) | theorem 560 | sorry 9 (all under `VeilTest/`: `Regression/TemporalBasic.lean` 2, `Regression/VerifierInteractiveRollback.lean` 2, `WarnTrustingSmtSolver.lean` 1, others) | admit 6 | native_decide 24 | implemented_by 8 | partial 61 | opaque 2. Dossier: `references/Veil-Lentil-Velvet-Loom`.
- Current README (read in full): "Veil is a framework for (1) specifying, (2) implementing, (3) model-checking, and (4) proving safety and liveness properties of state transition systems, with a focus on distributed protocols." Example list includes `Examples/Liveness/` ("Temporal properties proved with `prove_temporal_by`") and `Examples/LivenessMC/` ("model checked with `#temporal_check`, including TLA+ specifications and CAVA benchmarks").
- `docs/Distributed-DSL.md` (read): "For message-passing protocols there is a more compact way: describe the program each node runs, and the network that connects the nodes." Network options (verbatim table): `synchrony` async | sync; `loss` reliable | lossy; `channels` set | multiset; `order` unordered | fifo; `faults` none | `crash f` | `byzantine p`. "The compiled module is strict by construction: a node can only read and write its own variables, and can only learn about other nodes through messages."
- `docs/DSL-Reference.md` (read): temporal properties are "LTL formulas over the runs of the system" with `temporal [electsLeader] (forall n next, WF (send n next)) /\ ... -> diamond (exists l, leader l)`; `#temporal_check` "explores every infinite run of the instance, including runs that never take an enabled action, so liveness properties usually need fairness assumptions"; `prove_temporal_by` "proves a temporal property for all instantiations of the module, in the proof mode of Lentil ... It assumes the invariants (`hInv`) rather than proving them". Veil's temporal layer uses Lentil's syntax and rules (e.g. WF1). A liveness proof example is named `Examples/Liveness/TLA/Consensus.lean`. `Examples/Demo/FloodSet/FloodSetLiveness.lean` was downloaded: 128 lines, properties `terminates`, `decisionStable`, `everyoneDecides`, with `prove_temporal_by [decisionStable]` at line 102.
- CAV 2025 paper (https://link.springer.com/content/pdf/10.1007/978-3-031-98682-6_2.pdf, read in full text): abstract: "an open-source framework for automated and interactive verification of transition systems, aimed specifically at conducting machine-assisted proofs about concurrent and distributed algorithms"; "verification conditions in first-order logic, to be discharged automatically via a range of SMT solvers". RQ1 used "16 case studies": 9 from IvyBench, 2 from Padon et al. on Paxos, 5 others; "The case studies in our set total 1704 non-empty, non-comment lines of Lean code". "Veil verifies all but 2 benchmarks in under 15 s (87.5%), and all but 4 benchmarks in under 10 s (75%)." Soundness theorem: "forall s post, tr post s <-> (forall s r, tr s r s => post r s) ... for all actions tr with no failing assertions." Environment: Lean 4.16.0, cvc5 1.2.1, Z3 4.14.0. The CAV paper text contains no discussion of liveness proofs (liveness appears only in a cited title); the later README adds them (see C3).
- Liveness example read in full (`Examples/Liveness/TLA/Consensus.lean`): a PlusCal/Lentil+ "Tiny Consensus Specification" with `relation chosen : value -> Bool`, actions `choose`/`stutter`, `safety [agreement] chosen V1 /\ chosen V2 -> V1 = V2`, `set_option veil.smt.trust false`, then `temporal (only := verify) [success] WF choose -> diamond chosen-exists` and `prove_temporal_by [success]`. It is an abstract one-step spec, not a message-passing protocol.
- Further CAV paper facts (read from the PDF): "Veil verifies all but 2 benchmarks in under 15 s (87.5%)"; RQ2 covers four case studies outside the EPR fragment (including two Paxos variants from Padon et al.), where "SMT solvers use heuristics ... which may yield unknown". Related-work section states TLAPS "does not offer easy extensibility via meta-programming" and that Alloy is bounded. The invariant synthesizer `#updr` (docs/Synthesizer-Guide.md) "reports an inductive invariant, an execution that violates the properties, or that no universal inductive invariant exists", and with `certify` the result "becomes a theorem checked by the Lean kernel".

**S20. Veil, Loom and Dafny-style verifiers (Dafny 2026 workshop paper) [P][FULL] tags: H1, H2**
https://verse-lab.org/papers/veil-dafny26.pdf. Describes Veil's design and a Ring leader-election example with both automated and interactive proof (`theorem send_tr_single_leader'`); states external SMT solvers "do not have to be trusted: Veil supports proof reconstruction (at a 3-5x performance penalty)". No occurrences of "liveness" or "temporal" in the retrieved text.

**S21. Loom (POPL 2026) paper and repository [P][FULL+CODE] tags: H1, H2, H3**
https://verse-lab.org/papers/loom-popl26.pdf and https://github.com/verse-lab/loom | SHA 78928abc90 | 2026-02-21 | toolchain v4.24.0 | LICENSE present | 38 files, 9,628 LOC (7,068 code) | theorem 228 | partial 31 | axiom 1 (`Loom/SMT.lean:215`: `axiom trust_smt : forall (p : Prop), p`, used to close goals delegated to an SMT solver) | admit 1.
- Paper: "Loom: A Framework for Automated Generation of Foundational Multi-Modal Verifiers" (README title). Abstract mentions verifiers for "(1) distributed protocol safety and (2) Dafny-style specification" code; the Veil 2.0 benchmark suite "contains 17 specifications of 15 different distributed protocols". Section 7 "Combining Runtime and Deductive Verification of Distributed Protocols" describes a simulator for protocols "implemented using Lean's ..." (execution inside Lean, i.e. runtime checking of the model, not extraction to another language). Loom is built on "monadic shallow embedding of an executable program semantics into Lean 4", "Monad Transformer Algebras" for weakest preconditions. Case studies directory: Cashmere, Velvet.
- Velvet (verse-lab/velvet) counts only, SHA b2a349a570, 2026-11-05, toolchain v4.35.0: 140 files, 34,674 LOC, theorem 719, sorry 0, native_decide 22, partial 5.
- NOPaxos case study (paper section 7.2, read): "Our simplified model of the protocol omits these components [view change and synchronisation], so we consider only the case with a fixed leader." The authors "implemented a randomised simulation framework for Veil protocols as a monadic program and verified its soundness in Loom itself, proving that it produces only traces that are admitted by the specification." Inspecting simulator traces revealed that the transition `handle_gap_commit_reply` "appeared to never be taken", because "one of its require statements was always false"; they fixed it and "ultimately certified the safety property of the corrected specification". The simulator runs on the model, via "Lean's standard Gen monad ... using native Lean compiler"; no extraction of code to another language or comparison with a deployed NOPaxos implementation is described in these passages.
- Veil 2.0 benchmark: "17 specifications of 15 different distributed protocols (two protocols are verified both in a decidable fragment, and separately, in general first-order logic)". `Loom/SMT.lean:215` declares `axiom trust_smt : forall (p : Prop), p`; Veil's `veil.smt.trust false` option (used in the Consensus example above) is the documented switch for reconstruction mode, and `VeilTest/WarnTrustingSmtSolver.lean` tests the warning for trusting the solver.

**S22. raft-lean-squad (Rust raft-rs with Lean model) [P][FULL+CODE] tags: H2, H3, D**
https://github.com/dsyme/raft-lean-squad | SHA ab7a184fbd | last commit 2026-06-03 | LICENSE present | 80 Lean files, 21,355 LOC (8,726 code) | theorem 779 (token count; REPORT says 716, see C4) | sorry 0 | axiom 0 | native_decide 2. `formal-verification/lean/lean-toolchain` says v4.28.0 while the REPORT cites 4.30.0-rc2 (C4). Dossier: `references/raft-lean-squad`.
- REPORT quotes (fetched earlier in full, rechecked metadata): "716 theorems, 79 Lean files, 0 sorry"; models "not mechanically extracted from Rust source"; abstract u64 modeled as `Nat`; no "full message-passing network model", no liveness, no concurrency/IO. Correspondence checked by `#guard` assertions; "concrete function-level discharge" proposed as future work. The repo is produced by automated agents (".github/workflows/lean-squad.md", "agentic-workflows" files): flagged AI-generated on file evidence. Repository also contains `Charon.toml` (Aeneas front-end config) but the Lean models are hand-written per the REPORT.
- Count disagreement across the repo's own documents (C4): REPORT.md says 716 theorems and 79 Lean files; `CORRESPONDENCE.md` (updated 2026-04-27) says 681 theorems, 539+ `#guard` checks, 75 Lean files, 27 correspondence targets, with levels Exact, Abstraction, Approximation, Mismatch; `CRITIQUE.md` cites 769; the code scan at HEAD finds 779 `theorem`/`lemma` tokens in 80 files.

**S23. ConsensusVerifier ("The End of Consensus Research?" / AI-greement) [P][FULL] tags: H2, H3, D**
https://arxiv.org/html/2610.30114 (read earlier in full; no repository URL was found in the paper text). About 135K lines of Lean covering Multi-Paxos, FastPaxos, SwiftPaxos, EPaxos*, Bodega and A-greement; proves linearizability of protocol models; quote: "our Lean proofs cover a protocol model whose correspondence to the prototype is additionally checked by a human consensus expert"; "currently targets safety only". The authors distinguish formalizing from pseudocode versus implementation code; whether to formalize implementation behavior directly "remains an open question". Agent-generated proofs; kernel-checked.

**S24. verified-consensus (Lean 4, decoupled consensus) [P][CODE, counts only] tags: H2**
https://github.com/fradamt/verified-consensus | SHA d379754762 | 2026-10-12 | toolchain v4.34.1 | LICENSE present | 1,047 files, 334,092 LOC (256,931 code) | theorem 8,030 | sorry 0 | axiom 0 | native_decide 5 | partial 1. Content not read beyond counts. Dossier: `references/verified-consensus`.

**S25. lean-dag (Mysticeti-style DAG consensus) [P][CODE] tags: H2, D**
https://github.com/mlegner/lean-dag | SHA 243db27917 | 2026-09-07 | toolchain v4.32.2 | LICENSE present | 478 files, 78,475 LOC (43,782 code) | theorem 2,379 | sorry 0 | axiom 0.
- README provenance (verbatim): "Code and prose in this project were co-written with heavy LLM assistance. The Lean proofs are machine-checked ... but whether the definitions and theorem statements capture their intended meaning, ... has only human-plus-LLM review behind it. Read critically." Proved: safety "with no network assumption of any kind"; liveness "above eventual DAG synchrony, a structural condition on the DAG under which no liveness theorem mentions time"; "view convergence - after stabilisation, whatever one correct validator holds reaches every correct validator within Delta". Committee sizes `n >= 3f+1` (core), `n >= 5f+1`, `n = 3f + 2p - 1`, `n >= 5 fb + 3 fc + 1`, `n >= 2f+1` for crash only.
- Further README facts (read): liveness threshold "the constant `2Δ + proc`"; "the whole of what the network must supply reduces to a single clause of view convergence"; modules `LeanDag/Quality/`, `LeanDag/DoS/`, `LeanDag/GC/`; the pacemaker's catch-up rule bounds clock spread. A theorem `commits_recur_local` states that every reliable validator decides on its own view.

**S26. lean-verification-components / leanbook (Raft, Paxos, PBFT files) [P][CODE] tags: H2, D**
https://github.com/solanaqubits/leanbook | SHA 5276529c6c | 2026-11-05 | toolchain `leanprover/lean4:v4.33.1` | LICENSE present | 747 files, 165,863 LOC (137,886 code) | theorem 8,829 | sorry 0 | axiom 0 | extern 2 | opaque 2. The repo is a general mathematics-and-verification Lean book; the consensus part is `Verification/DistributedRaft*.lean` (20 files, 10,376 lines), `DistributedPaxos*`, `DistributedPBFTConsensus`, `BFTConsensusQuorum`.
- `DistributedRaftConsensus.lean`: `theorem election_safety_unique_leader ... (hA_votes : forall x in A, votedFor x = c1) (hB_votes : forall x in B, votedFor x = c2) : c1 = c2`, proved from majority-quorum overlap (`majority_intersection_nonempty`). The repo's own comment on `leader_commit_quorum_overlap`: "Set overlap only, without a log-freshness rule or a fault model." `DistributedRaftLeaderCompleteness.lean` (header read; author tag "aicyberg") is titled "Conditional Raft Leader Completeness": "Log Matching, entry provenance, election restrictions and the admissibility of voter histories are explicit premises. They are not proved invariants of network...". `knowledge/SUMMARY.en.md` (line 29) says "The network-induction module now derives global Log Matching and historical entry/message provenance from those transitions. The complete-bridge module derives Leader Completeness...". The two statements describe different modules in the same repo and were not reconciled by running them. The repo has an automated-update process described in its docs; flagged likely AI-generated on that basis. No link to Raft implementation code was found.

**S27. tla-plus-lean-workflow (STTT submission artifact) [P][CODE] tags: H3, D**
https://github.com/akkeshavan/tla-plus-lean-workflow | SHA 9910342183 | 2026-06-29 | no toolchain file at root | no licence file | 20 Lean files, 3,377 LOC | theorem 69 | sorry 0 in the token scan, but source comments say "All proof obligations are marked `sorry` pending Veil/SMT discharge" (`VerificationContinuum/OpenRaft/Propositional.lean:10`) and "two sorry stubs" (`VeilProofs/PaxosVeil.lean`, `OpenRaftVeil.lean`), so the scan and comments disagree (the stubs are described in comments or Veil-side files not matched by the token scan). Pipeline: "Phase 1 TLA+ model checking; Phase 2 Lean 4 functional model; Phase 3 Lean 4 propositional proofs (inductive invariants via Veil/SMT); Phase 4 Rust implementation (Aeneas functional translation where applicable); Phase 5 Solarkraft runtime monitoring". Six case studies (two-phase commit, token mutex, Raft election, openraft, Paxos, Lightning). Author: single independent researcher.
- Read from the repo: Rust source directories exist for the token-mutex case study (`case-studies/token-mutex/rust/src/lib.rs` and `lib_bug_tm4.rs`, a seeded-bug variant); other case studies list Rust only in the layout description. The openraft Lean file states "All proof obligations are marked `sorry` pending Veil/SMT discharge" while the repo-wide scan finds no `sorry` token (C6). The author is an independent researcher; the paper is a submission, not yet published.

**S28. compositional-temporal-safety [P][CODE] tags: H2**
https://github.com/repowazdogz-droid/compositional-temporal-safety | SHA 7a812bdb46 | 2026-07-20 | LICENSE present | 7 Lean files, 481 LOC (267 code) | theorem 19 | sorry 0. README: assume-guarantee composition of per-agent invariants for a roster of agents sharing a budget and a critical-section token; proves `totalSpent rs s.spent <= cap` and mutual exclusion "at every reachable state" for "an arbitrary finite Nodup roster". Claims are mapped to "evidence grades" in the repo. The README says the artefact is a verification artefact for a toy multi-agent system and names unclosed gaps shared with sibling repos.

### 1F. Concurrency models, separation logic, Lean runtime

**S29. Iris-Lean (Iris in Lean 4) [P][CODE] tags: H1, H2**
https://github.com/leanprover-community/iris-lean | SHA 8293092f09 | 2026-11-05 | LICENSE present | 324 files, 100,399 LOC (74,181 code) | theorem 5,454 | axiom 5 (all in `Iris/IrisTest/Benchmarking/BenchmarkHeapLangList.lean`: `isList_nil`, `isList_cons`, `nil_spec`, `cons_spec`, `length_spec`) | sorry 0 | opaque 3 | partial 46. HeapLang is under `Iris/IrisTest/HeapLang/` (WeakestPre, HeapTactics, Apply, Notation).

**S30. "Iris in Lean" paper (arXiv 2609.24252) [P][FULL] tags: H1, H2**
https://arxiv.org/pdf/2609.24252. Describes the Lean port of Iris (ghost resources, step-indexed BI, proof mode) and a port of HeapLang (section 2.2). Quote from the introduction: "Iris-Lean ... ported more than 80% of the separation logic for the HeapLang programming language, including a variety of examples." The port paper records limitations of the proof-mode rewriting ("does not yet support rewriting by single entailments"). It does not present a distributed protocol verification.

**S31. LeanIn (concurrency runtime with a bridge from model to runtime) [P][CODE] tags: H1, H3, D**
https://github.com/scidonia/leanin | SHA 3015043d80 | 2026-10-31 | toolchain v4.35.0 | LICENSE present | 78 files, 21,376 LOC (13,183 code) | theorem 272 | axiom 11 (`LeanIn/Theory/Bridge.lean` 9, `LeanIn/Test/BridgeControls.lean` 2) | opaque 23 | partial 24 | extern 20 | sorry 0.
- README: "a concurrency runtime with a work-stealing scheduler for Lean 4 ... on par with other highly concurrent languages but which lives in a proof assistant." Benchmark table compares it with Node 22 and the shipped `Std.Http.Server` (for example "depth 1 | shipped Std.Http.Server | 5 323 | 5 584 | 7 626" requests/s against leanin "62 568 | 147 791 | 126 481").
- `Bridge.lean` header (verbatim): "The bridge: the only axioms. `World.lean` proves the model's properties - mutual exclusion, no notification memory, permitted spurious wakeups, no fairness. None of that is assumed. What is assumed is the connection between that model and the running program ... Lean gives `IO`/`BaseIO` no semantics (`docs/proof-strategy.md` Gap A), so the statement 'running this operation advances the world like that' cannot be derived - it has to be named." Nine operation contracts (mutex create/lock/unlock, condvar create/wait/notify, clock) form the TCB.
- Companion document `docs/lean-concurrency-today.md` (read in full): table of Lean's concurrency surface (Task, Promise, `Std.Async`, `Selector`, timers, TCP/UDP, HTTP server); gap table row "proofs | none, anywhere | build (the actual research)"; `race` "does not cancel the loser".

**S32. Lean reference manual: Logical Model of IO, Tasks and Threads, Asynchronous Programming [P][FULL] tags: H1, H3**
https://lean-lang.org/doc/reference/latest/IO/Logical-Model/ and https://lean-lang.org/doc/reference/latest/IO/Async/. Logical-model page: "Side effects, which must be executed in the correct order, are abstractly described in Lean's logic. When programs are run, the Lean runtime system is responsible for actually carrying out the described effects." `IO alpha` "can be thought of as a state monad in which the state is the entire world." Async page: "The Async monad provides tools and abstractions for constructing asynchronous programs that can safely multiplex different sources of data. Typical use cases include network servers and other interactive applications." Neither page gives a formal semantics of concurrent interleaving or a memory model.

**S33. Std.Http in Lean core (HTTP/1.1 protocol state machine) [P][CODE] tags: H1, H3**
https://github.com/leanprover/lean4/tree/master/src/Std/Http (sparse clone of `src/Std/Http` and `tests/elab` at SHA a2f1a642ab, 2026-11-06). 47 files, 13,726 LOC (6,322 code) | theorem 24 | partial 21 | sorry 0. `Protocol/H1.lean` header: "HTTP/1.1 Protocol State Machine ... implements the core HTTP/1.1 protocol state machine that handles parsing requests/responses and generating output. The machine is direction-aware, supporting both server mode ... and client mode". The 24 theorems are in data-helper files (`Data/URI/Encoding.lean` 13, `Internal/LowerCase.lean` 5, `Internal/Char.lean` 2, `Internal/MultiMap.lean` 2, others); no theorem files exist under `Protocol/`. Tests: `tests/elab/http_multimap.lean`, `async_http_query_encoding.lean`, `timeHttpDate.lean`. Authors: Sofia Rodrigues (Lean FRO). Dossier: `references/Std.Http`.

### 1G. Linking models to implementations

**S34. Cedar paper "How We Built Cedar: A Verification-Guided Approach" [P][FULL] tags: H3, D**
https://arxiv.org/pdf/2407.01688. Describes "verification-guided development" (VGD): Lean models written to be "used as oracles for differential testing" and "differential random testing [29] to increase our confidence that the behavior of our formal models matches that of our production" Rust code. Properties proved in Lean include "Forbid trumps permit". The model-to-code link is by differential random testing, per the paper's own wording.

**S35. cedar-spec repository [P][CODE counts] tags: H3**
https://github.com/cedar-policy/cedar-spec | SHA a4c5a89cd1 | 2026-11-06 | toolchain v4.35.0 | LICENSE present | 388 files, 126,845 LOC (102,084 code) | theorem 2,600 | sorry 0 | axiom 0 | native_decide 26 | implemented_by 2 | opaque 2 | partial 42. FLoC 2026 slides by de Moura (https://leodemoura.github.io/static/floc26/, read): "Executable Lean model alongside Rust production code. Lean model ~10x smaller than Rust"; "~100M differential random tests nightly. Lean: 5 us/test. Rust: 7 us/test"; "Release gate: no Cedar version ships unless model, proofs, and differential tests are current." AWS blog (https://aws.amazon.com/blogs/opensource/lean-into-verified-software-development/, read): "Over the past 6 months, we built a formal model of Cedar in Lean, and proved that Cedar's components satisfy key safety and security properties." [S] for the blog. Cedar is an authorization-policy evaluator, not a distributed protocol. Dossier: `references/Cedar`.
- Repo structure (README): `cedar-lean` ("Lean formalization of, and proofs about, Cedar"), `cedar-drt` ("fuzzing, property-based testing, and differential testing"), `cedar-policy-generators`, `cedar-lean-ffi` ("Rust bindings for interacting with the Lean formalization of Cedar"), `cedar-lean-cli`. `cedar-lean/Cedar/` includes `Spec`, `Thm`, `Validation`, `SymCC` (symbolic compilation, needs cvc5), `TPE`. README: "the formalization of Cedar and infrastructure for performing differential randomized testing (DRT) between the formalization and Rust production implementation".

**S36. Aeneas (Rust to Lean functional translation) [P][CODE+FULL] tags: H3, H4**
https://github.com/AeneasVerif/aeneas | SHA c5eb4a28ce | 2026-11-07 | LICENSE.md present. README lines 124 and 138-139 (read): Aeneas does not "support unsafe code and concurrent code"; unsupported features listed: "unsafe code", "concurrency". `docs/user/src/README.md:16`: "no concurrent execution: long-term effort. We plan to address coarse-grained parallelism as a long-term goal." Aeneas project page (https://aeneasverif.github.io/projects/): Charon "acts as an interface between the rustc compiler and program verification projects ... operates on MIR ... converts MIR code to ULLBC", with JSON serializers. Aeneas paper (https://arxiv.org/pdf/2206.07185, ICFP 2022) abstract: "we tweak our semantics to abstract the aliasing graph in the presence of function calls ... backward functions, which obviates the need for prophecy variables"; backends in the paper: F* and a Coq backend in the works (the paper predates the Lean backend). Dossier: `references/Aeneas-SymCrypt`.

**S37. SymCrypt-in-Rust verification with Aeneas (arXiv 2609.15648, draft report) [P][FULL] tags: H3**
https://arxiv.org/pdf/2609.15648. Authors Ho, Fournet, Protzenko, Naehrig and others; "We target production code written in Rust ... Rust's ownership discipline enables Aeneas to extract a pure model of this code in Lean". Cryptographic primitives, sequential code; no protocol-level model. Dossier: `references/Aeneas-SymCrypt`.

**S38. Federated verification architecture (Bullish) (arXiv 2606.02019) [P][FULL] tags: H3, D**
https://arxiv.org/pdf/2606.02019. "We propose a federated architecture for production formal verification" with TLA+ obligations discharged "by citing an equivalent theorem in a structurally distinct kernel" (Coq `Print Assumptions`, Lean 4 `#print axioms`); "agreement is build-system-enforced rather than a logical soundness theorem; the residual correspondence-certificate question is named as an open problem in section 6.3". Financial-systems case; Lean is one of several kernels.

### 1H. Security protocols (symbolic model)

**S39. DyLean repository and Bob DyLean paper [P][CODE] (paper: abstract page only, PDF not retrievable here) tags: H1, H2, H3, D**
https://github.com/BobDyLean/dylean | SHA 1662f9ad82 | 2026-11-06 | toolchain v4.36.0-rc1 | LICENSE present | 98 files, 26,319 LOC (21,464 code) | theorem 597 | sorry 34 (`Tests/Meta/Step.lean` 29, `DY/Meta/LetUtils.lean` 5; read: these are test stubs such as `def hash (b: Bytes): Bytes := sorry` under `set_option warn.sorry false`, i.e. placeholders in test/meta code, not proofs in `Examples/`) | admit 1 | native_decide 14 | opaque 1. Paper-artifact repo https://github.com/BobDyLean/bobdylean-paper-artifact SHA e24fe13d68, 2026-07-17, toolchain v4.32.0, 95 files, 26,167 LOC. Paper (https://eprint.iacr.org/2026/1493, Wallez and Cremers, published 2026-07-21, not peer reviewed per the source's trust note): "DyLean, a framework for the symbolic analysis of cryptographic protocols in the Lean theorem prover ... comprises both a flexible general-purpose symbolic semantics, as well as a concrete proof methodology"; "based on trace invariants"; case studies "merkle trees, ratcheting protocols, post-quantum protocols, and protocols analyzed under different equational theories". README disclaimer: "We are actively working on DyLean, expect breaking changes." Example directories: `ACME`, `MerkleTree`, `Ratchet`, `SignedDH`, `SignedDHKEM`. Symbolic model: attacker may only use cryptographic functions as black boxes (Wallez talk slides S40).

**S40. Wallez, "Proofs of Security Protocols in Lean" (Google talk, Dec 2025) [P][FULL] tags: H1, D**
https://www.twal.org/pdf/slides/2025_12_google.pdf. Slides state "still an early work-in-progress"; the symbolic attacker "can run unbounded instances of the protocol in parallel" with "Only limitation of the attacker: only allowed to use cryptographic functions as black boxes. This captures all 'logical attacks'." DY* is written in F* with Z3; DyLean is the "next iteration".

### 1I. Lean 3 lineage

**S41. unitb/temporal-logic (Lean 3) [P][CODE] tags: H1, H2**
https://github.com/unitb/temporal-logic | SHA accec04d1b | last commit 2018-11-01 | no licence file | `leanpkg.toml` pins Lean 3.4.1 | 17 files, 6,477 LOC | sorry 2 (raw grep, may include comments) | axiom 0. Source tree: `src/temporal_logic/{basic, fairness, feasibility, lemmas, persistent, scheduling, spec, tactic, refinement/...}`. `fairness.lean` defines `wf : cpred := diamond box p -> box diamond A`, `sf : cpred := box diamond p -> box diamond A` and a `sched` combinator. Zulip summary lists it as "LTL + refinement".

**S42. unitb/unitb-semantics (Lean 3) [P][CODE] tags: H2, D**
https://github.com/unitb/unitb-semantics | SHA 07607ddb2c | last commit 2018-01-04 | 29 files, 7,284 LOC | sorry 3 (raw grep) | axiom 0. GitHub description: "Proof of soundness of the Unit-B refinement calculus". README: "Using Lean to formally verify the proof obligations of Literate Unit-B; verify implementation rules for Unit-B specifications; certify proofs created by Literate Unit-B". Directories: `category, code, decomposition, logic, models, refinement, scheduling`. Method paper: "The Unit-B Method - Refinement Guided by Progress Concerns" (https://arxiv.org/abs/1810.10143, read): "a formal method inspired by Event-B and UNITY ... stepwise design of software systems satisfying safety and liveness properties ... a refinement order preserving both liveness and safety properties."

**S43. GaloisInc/lean-protocol-support (Lean 3) [P][CODE] tags: H1, H3**
https://github.com/GaloisInc/lean-protocol-support | SHA cabfa3abed | last commit 2017-09-21 | LICENSE present | 95 files, 15,862 LOC | sorry 16 (raw grep) | axiom 0. Directories `galois/temporal` (`LTS.lean`, `temporal.lean`, `fixpoint.lean`, `classical.lean`), `galois/network` (`network_monad.lean`, `network_implementation.lean`, `network_local_abs.lean`, `action.lean`, `agent_facts.lean`, `labels.lean`), `galois/csp`. `network_monad.lean` header (author Ben Sherman): "a class `network_monad` that extends monad with operations for communicating over sockets. These sockets can send and receive arbitrary length byte sequences as messages"; timing model "assumes that all computation including sending and receiving messages takes zero time". `temporal/LTS.lean`: "A labeled transition system built on top of temporal logic".

**S44. loganrjmurphy/lean-temporal (Lean 3) [P][CODE] tags: H1**
https://github.com/loganrjmurphy/lean-temporal | SHA 40c6ad1502 | last commit 2020-11-11 | `leanpkg.toml` Lean 3.23.0 | 2 files, 364 LOC | sorry 4 (raw grep). README: "Formalization of (some) Temporal Logics in Lean ... Definitions used are from Baier and Katoen, Principles of model checking."

**S45. Lean Zulip: "Temporal Logic" thread (Sep to Oct 2023) [P][FULL] tags: D**
https://leanprover-community.github.io/archive/stream/217875-Is-there-code-for-X%3F/topic/Temporal.20Logic.html. A poster lists "projects ... about reasoning with concurrent/distributed systems in Lean": iris-lean (higher-order concurrent separation logic), `lean-temporal` (LTL + CTL), `unitb/temporal-logic` (LTL + refinement), `GaloisInc/lean-protocol-support` (LTL + LTS); quote: "Actually, only the last three match the topic, and they're all in Lean 3." Simon Hudon (author of unitb): "It's a bit old. I've been meaning to keep it up to date with mathlib and lean but I haven't had time in a while."

**S46. Hudon, "Embedding specialized proof languages into Lean: A Case Study" (Lean Together 2019) [P][FULL] tags: H1, D**
https://lean-forward.github.io/lean-together/2019/slides/hudon.pdf. Slides on embedding temporal-logic proof notation (Unit-B style) in Lean 3 via tactic and notation overrides ("override tactic notation (thanks to Lean!)", "define top-level syntax (thanks to Lean!)"), with code references into `src/temporal_logic/lemmas.lean`.

### 1J. Non-Lean comparators (background only)

**S47. Lamport, "Specifying Systems" chapter (spec-book-chap.pdf) [P][FULL] comparator; tags: D**
https://lamport.azurewebsites.net/pubs/spec-book-chap.pdf. Used only to record how TLA+ itself defines specification/safety/liveness terminology (not Lean).

**S48. Ongaro and Ousterhout, Raft paper [P][FULL] comparator; tags: D**
https://raft.github.io/raft.pdf. Used only as the reference protocol description behind Raft models in S22, S26, S27.

## 2. Repository metrics table

| ID | Repo | SHA | Toolchain | Last commit | Lean LOC | theorem | sorry | axiom | native_decide | implemented_by | extern | opaque | partial |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| S1 | lean-machines | 9f1f001f0a | v4.35.0-rc3 | 2026-11-04 | 11,252 | 23 | 10 | 4 (5 by grep) | 0 | 0 | 0 | 0 | 0 |
| S2 | lean-machines-examples | 4544600e38 | v4.35.0-rc3 | 2026-11-06 | 9,637 | 227 | 25 | 2 | 0 | 0 | 0 | 0 | 0 |
| S4 | leslie | 58e58b2502 | v4.27.0 | 2026-06-09 | 103,805 | 2,216 | 5 | 0 | 1 | 0 | 0 | 0 | 14 |
| S5 | lean-tla | 99d8be4103 | v4.33.0-rc2 | 2026-08-17 | 141,774 | 3,404 | 5 (vendored) | 0 | 7 | 0 | 0 | 0 | 50 |
| S6 | Lentil | 1ddf1fc161 | v4.35.0 | 2026-11-06 | 12,004 | 318 | 0 | 0 | 0 | 0 | 0 | 2 | 16 |
| S8 | LeanLTL | 56ca11b3d6 | v4.17.0-rc1 | 2026-10-20 | 4,408 | 309 | 14 | 2 | 0 | 0 | 0 | 0 | 3 |
| S9 | LeanearTemporalLogic | 24796b51e0 | v4.18.0-rc1 | 2025-07-27 | 4,244 | 70 | 15 | 0 | 0 | 0 | 0 | 0 | 0 |
| S10 | zero-to-qed | cf59ccfb80 | v4.34.1 | 2026-11-01 | 9,045 | 405 | 3 | 2 | 7 | 0 | 0 | 1 | 5 |
| S11 | cslib | 76b7943d0b | v4.36.0-rc2 | 2026-11-05 | 52,113 | 3,050 | 0 | 0 | 0 | 0 | 0 | 2 | 0 |
| S15 | telltale | c8c6a7fb37 | v4.35.0 | 2026-10-31 | 164,896 | 5,101 | 0 | 0 | 0 | 8 | n/r | 0 | 1 |
| S16 | rumpsteak-aura | 024b769afe | n/r | 2026-01-30 | 58,185 | 2,287 | 0 | 35 | 2 | 0 | n/r | 0 | 5 |
| S19 | veil | 57447722e7 | v4.35.0 | 2026-11-07 | 89,474 | 560 | 9 | 0 | 24 | 8 | n/r | 2 | 61 |
| S21 | loom | 78928abc90 | v4.24.0 | 2026-02-21 | 9,628 | 228 | 0 | 1 | 0 | 0 | n/r | 0 | 31 |
| S21 | velvet | b2a349a570 | v4.35.0 | 2026-11-05 | 34,674 | 719 | 0 | 0 | 22 | 0 | n/r | 0 | 5 |
| S22 | raft-lean-squad | ab7a184fbd | v4.28.0 (REPORT: 4.30.0-rc2) | 2026-06-03 | 21,355 | 779 | 0 | 0 | 2 | 0 | n/r | 0 | 0 |
| S24 | verified-consensus | d379754762 | v4.34.1 | 2026-10-12 | 334,092 | 8,030 | 0 | 0 | 5 | 0 | n/r | 0 | 1 |
| S25 | lean-dag | 243db27917 | v4.32.2 | 2026-09-07 | 78,475 | 2,379 | 0 | 0 | 0 | 0 | n/r | 0 | 0 |
| S27 | tla-plus-lean-workflow | 9910342183 | n/a | 2026-06-29 | 3,377 | 69 | 0 (comments mention sorry stubs) | 0 | 0 | 0 | n/r | 0 | 0 |
| S26 | leanbook | 5276529c6c | v4.33.1 | 2026-11-05 | 165,863 | 8,829 | 0 | 0 | 0 | 0 | 2 | 2 | 0 |
| S28 | compositional-temporal-safety | 7a812bdb46 | n/a | 2026-07-20 | 481 | 19 | 0 | 0 | 0 | 0 | n/r | 0 | 0 |
| S29 | iris-lean | 8293092f09 | n/r (in Iris/) | 2026-11-05 | 100,399 | 5,454 | 0 | 5 (benchmark file) | 0 | 0 | n/r | 3 | 46 |
| S31 | leanin | 3015043d80 | v4.35.0 | 2026-10-31 | 21,376 | 272 | 0 | 11 | 0 | 0 | 20 | 23 | 24 |
| S35 | cedar-spec | a4c5a89cd1 | v4.35.0 | 2026-11-06 | 126,845 | 2,600 | 0 | 0 | 26 | 2 | n/r | 2 | 42 |
| S39 | dylean | 1662f9ad82 | v4.36.0-rc1 | 2026-11-06 | 26,319 | 597 | 34 | 0 | 14 | 0 | n/r | 1 | 0 |
| S33 | lean4 src/Std/Http | a2f1a642ab (master) | n/a | 2026-11-06 | 13,726 | 24 | 0 | 0 | 0 | 0 | n/r | 0 | 21 |
| S41 | unitb/temporal-logic (Lean 3) | accec04d1b | Lean 3.4.1 | 2018-11-01 | 6,477 | n/r | 2 (raw) | 0 | n/a | n/a | n/a | n/a | n/a |
| S42 | unitb/unitb-semantics (Lean 3) | 07607ddb2c | Lean 3 (master) | 2018-01-04 | 7,284 | n/r | 3 (raw) | 0 | n/a | n/a | n/a | n/a | n/a |
| S43 | GaloisInc/lean-protocol-support (Lean 3) | cabfa3abed | n/r | 2017-09-21 | 15,862 | n/r | 16 (raw) | 0 | n/a | n/a | n/a | n/a | n/a |
| S44 | loganrjmurphy/lean-temporal (Lean 3) | 40c6ad1502 | Lean 3.23.0 | 2020-11-11 | 364 | n/r | 4 (raw) | 0 | n/a | n/a | n/a | n/a | n/a |

n/r = not recorded (the clone was removed under the disk limit before this field was captured, or the pattern was not in the count script for that run). "raw" = plain `grep -w sorry`, may include comments. Lean 3 repos were not scanned with the Lean 4 comment-stripping script.

## 3. Evidence by sub-claim (side by side, no verdict)

### H1 Expressiveness (state machines, message passing, concurrency, nondeterminism, adversary, time, probability)

| Evidence pointing toward limits | Evidence pointing toward expressibility |
|---|---|
| LeanMachines machines carry only state invariants; no network, temporal or fairness module (S1). | Veil `network (synchrony := async) (loss := lossy) (order := fifo) (faults := byzantine p)` compiles per-node programs and a network into a Lean module (S19). |
| Lean has no formal semantics for `IO`/`BaseIO` or concurrent interleaving; LeanIn: "Lean gives `IO`/`BaseIO` no semantics" (S31, S32). | Lentil defines `always`, `leads_to`, `weak_fairness`, `strong_fairness` over traces (S6); LeanLTL covers infinite and finite traces (S7). |
| lean-protocol-support (Lean 3) assumed a zero-time computation model (S43). | CSLib has LTS, CCS, HML, bisimulation, FLP and consistency specifications (S11, S14). |
| CSLib white paper lists concurrency and low-level reasoning as planned work (S12). | Telltale: asynchronous buffered multiparty session types with 5,101 theorems and no `sorry`/`axiom` in the scan (S15). |
| Probability: Leslie has a `Prob/` directory with a spike containing one `sorry` (S4). | Iris-Lean provides concurrent separation logic with HeapLang (S29, S30). |
| Real time: Veil, Lentil and LeanLTL discussed here are untimed (S6, S7, S19). Dolev-Yao attacker is symbolic, black-box crypto (S39, S40). | DyLean expresses protocols with unbounded loops and user-defined equational theories and threat models (S39). |

### H2 Provability (invariants, liveness, refinement, security)

| Evidence pointing toward limits | Evidence pointing toward provability |
|---|---|
| Several consensus developments exclude liveness: raft-lean-squad, ConsensusVerifier ("currently targets safety only") (S22, S23). | Veil proves temporal properties with `prove_temporal_by` using Lentil rules (WF1); FloodSetLiveness (S19). |
| Veil's `prove_temporal_by` "assumes the invariants (`hInv`) rather than proving them" (S19). | lean-dag proves safety and liveness "above eventual DAG synchrony" (S25); lean-tla proves a Streamlet leads-to under explicit schedule hypotheses (S5). |
| Leslie README lists `sorry` for Paxos and VRViewChange; code has 2 in LastVoting and 2 in Refinement (S4). | Leslie 2,216 theorems; lean-tla 3,404; verified-consensus 8,030 with no `sorry`/`axiom` in the scan (S4, S5, S24). |
| leanbook Raft theorem is "Set overlap only, without a log-freshness rule or a fault model" (S26). | CSLib proves FLP, a statement over all algorithms in the model (S11). |
| FLP-style results are impossibility results for the model class, not specific protocols (S11). | Veil CAV paper: 16 case studies verified; 87.5% under 15 s (S19). |
| DyLean repo has 34 `sorry` tokens in tests/meta (S39). | Telltale reports 0 `sorry`/`axiom` over 830 files (S15). |
| Large agent-generated developments are self-reported; no third-party audit located (S22, S23, S25, S26). | lean-dag states kernel-checked theorems and warns about statement fidelity (S25). |
| Rumpsteak-aura Lean `DeadlockFreedom.lean` uses 7 axioms (S16). | Lean 3 Unit-B soundness of a refinement calculus preserved both safety and liveness (S42). |

### H3 Implementation link

| Evidence pointing toward limits | Evidence pointing toward a link |
|---|---|
| Cedar link is differential random testing (S34, S35). | Telltale claims a "theorem-defined Rust<->Lean core protocol-machine runtime correspondence" (S15). |
| raft-lean-squad: models "not mechanically extracted from Rust source"; `#guard` checks (S22). | Aeneas extracts a pure Lean model from Rust and proofs are about that model (S36, S37). |
| ConsensusVerifier: model-to-prototype correspondence "additionally checked by a human consensus expert" (S23). | Loom/Veil: programs and simulators executed inside Lean; verifiers generated for imperative Dafny-style code (S19, S21). |
| Aeneas lists concurrency and unsafe code as unsupported (S36). | tla-plus-lean-workflow Phase 4: Rust "Aeneas functional translation where applicable" (S27). |
| Telltale TCB: "Rust runtime implementation ... comparison-checked, not fully proved" (S15). | LeanIn names its nine runtime-contract axioms and states that is the whole model-to-runtime connection (S31). |
| Zero-to-QED: "evidence of correspondence, not a proof for all inputs" (S10). | Std.Http places an HTTP/1.1 state machine in Lean core (S33); proofs about it were not found. |
| Federated paper: correspondence-certificate question "named as an open problem" (S38). | SymCrypt: production Rust code verified via Aeneas (sequential, crypto primitives) (S37). |

### H4 Generality ("any codebase")

| Evidence pointing toward limits | Evidence pointing toward routes |
|---|---|
| Aeneas: Rust only; no unsafe, no concurrency (S36). | Charon exposes MIR/ULLBC as JSON for other consumers (S36). |
| Cedar and Zero-to-QED links rely on test oracles per project (S10, S34). | CSLib/Boole plans verification conditions for mainstream-language code (S12, planned). |
| Telltale's claim is bounded to a "declared shipped surface" (S15). | Loom produces multi-modal verifiers for Dafny-style imperative programs (S21). |
| Leslie's Rust verification doc is a design sketch marked under review, and the Lean `Leslie/Rust` files listed in the README were not in the tree (S4, C2). | lean-protocol-support (Lean 3) targeted network programs through a monad interface (S43). |

### H5 Inference (protocol-level incapacity implies implementation-level incapacity)

| Evidence relevant to the "therefore" link | Counter-direction evidence |
|---|---|
| Where no protocol-level temporal proofs exist, no implementation-level claims were found (for example Std.Http: 24 theorems, none on the state machine) (S33). | Cedar's model-to-code link (differential testing) does not use protocol-level proofs; Zero-to-QED likewise (S10, S34). |
| Veil, Leslie, lean-tla prove protocol models, but those repositories do not state a mechanical implementation link (S4, S5, S19). | Aeneas-based Rust verification (SymCrypt) proves code properties without a protocol model (S37). |

### D Definitions: how sources use "verify" and "check"

- "Verified" = kernel-checked theorem about a Lean model: LeanMachines, Leslie, lean-tla, lean-dag, verified-consensus, Veil (S1, S4, S5, S25, S19).
- "Formally verified for the declared shipped surface" with a TCB ledger: Telltale (S15).
- "Verification-guided development" = proofs of a model plus differential random testing against production code: Cedar (S34).
- "Model checking" inside Lean: Veil `#model_check`, `#temporal_check` (finite instances) versus `prove_temporal_by` (all instances) (S19).
- "Evidence of correspondence, not a proof": Zero-to-QED (S10).
- "Checked by a human consensus expert": ConsensusVerifier (S23).
- "Operational conformance" versus "mechanized proof": Telltale digest/heap parity (S15).
- Lean 3 Unit-B: "certify proofs created by Literate Unit-B", i.e. Lean as a proof checker for an external tool (S42).

## 4. Conflicts (both sides, unresolved)

- C1. Leslie `sorry` status. README table: "Paxos ... Spec complete, 2 sorry", "VRViewChange ... 1 sorry". `Examples/Paxos/PaxosRefinement.lean` docstring: "abstract-ahead, 1 sorry". `Examples/Paxos/PaxosEvaluation.md`: "All proofs are machine-checked and sorry-free." Code scan at SHA 58e58b2502: 5 `sorry` tokens in total, in `Refinement.lean` (2), `Examples/LastVoting.lean` (2) and `Prob/Spike/ASTSanity.lean` (1); none in Paxos files; prose mentions of sorry also in `BenOr.lean` and `BallotLeaderPhased.lean` with no token. Four sources, three different statements; not reconciled (documents may be stale; no build was run).
- C2. Leslie Rust verification. The README file tree lists `Leslie/Rust/CoreSemantics.lean` and `RuntimeSemantics.lean`; a tree listing at SHA 58e58b2502 shows no such files (only `docs/rust-verification*.md` and the `rust/` crate). The design doc presents a `rust_correctness` theorem as a sketch and the plan doc is marked "UNDER REVIEW"; whether the Lean side exists elsewhere is unknown.
- C3. Veil liveness. Earlier materials (Phase 1 reading of the README at that time, the CAV 2025 and Dafny 2026 paper texts) describe safety verification and have no liveness section; the README at SHA 57447722e7 says Veil handles "safety and liveness properties" and documents `#temporal_check` and `prove_temporal_by`. Both are recorded; the later README is more recent. The Phase 1 version of this file recorded Veil as "safety-only" and is superseded by this entry.
- C4. raft-lean-squad numbers and toolchain. REPORT: "716 theorems, 79 Lean files", Lean 4.30.0-rc2; `formal-verification/CORRESPONDENCE.md` (last updated 2026-04-27, commit 27abfa4) reports "681 theorems, 539+ `#guard`, 75 Lean files, 27 correspondence targets" and defines correspondence levels Exact, Abstraction, Approximation, Mismatch; repo scan: 80 files and 779 `theorem`/`lemma` tokens; `lean-toolchain` v4.28.0. Differences may be due to document staleness or counting method.
- C5. CSLib and Raft. A secondary source seen in Phase 1 claimed a Raft port in CSLib; no Raft content exists in the cloned CSLib tree (SHA 76b7943d0b). Recorded as unconfirmed; the claim is removed from evidence.
- C6. tla-plus-lean-workflow `sorry`. The token scan finds 0 `sorry`, while source comments say proof obligations are "marked `sorry` pending Veil/SMT discharge" and mention "two sorry stubs". Not reconciled.
- C7. Telltale vs rumpsteak-aura, and Telltale internal. Same author; Telltale scan has 0 axioms in 830 files and its `lean/CODE_MAP.md` says "The repository is currently axiom-free", yet the same file's table still lists files with "2 axioms" and "8 axioms"; rumpsteak-aura has 35 axioms in 221 files, including `deadlock_free` and `not_stuck`. The relationship between the two repositories is not stated in the files read. Telltale's own distributed-family table marks Nakamoto and Reconfiguration as `placeholder` ("some exported guarantee is vacuous or backed by True-style assumptions") while its top-level claim is "formally verified for the declared shipped surface".
- C8. Iris-Lean location of HeapLang. The Iris-Lean paper says the authors "ported more than 80% of the separation logic for the HeapLang programming language, including a variety of examples"; in the repository HeapLang sits in `Iris/IrisTest/HeapLang` (a test-named directory) and the only `axiom` declarations are in a benchmark file. Not clear whether HeapLang is considered core or demonstration.
- C9. leanbook Raft: a file titled "Conditional Raft Leader Completeness" lists Log Matching and election restrictions as "explicit premises ... not proved invariants of network...", while the repo's summary says its network-induction module "derives global Log Matching" and a complete-bridge module "derives Leader Completeness". Not reconciled.

## 5. Gaps closed and gaps remaining (against the Phase 1 report)

Closed in Phase 1b:
- Veil liveness status read from the current README/docs rather than inferred (C3).
- CSLib protocol content inspected in the tree: FLP proof and consistency specifications found; Raft port not found (C5).
- Lean 3 lineage located and inspected: unitb/temporal-logic, unitb-semantics, GaloisInc/lean-protocol-support, lean-temporal, Zulip thread, Hudon slides (S41-S46).
- Telltale and rumpsteak-aura TCB statements read.
- Lean runtime/concurrency documentation read (S31, S32).

Also closed in the final pass: Zero-to-QED numbers (83,300 and 2,025) and limitation quotes re-verified against the chapter text; rumpsteak-aura `deadlock_free` and `not_stuck` confirmed as `axiom` declarations in source; DyLean `sorry` tokens identified as test stubs; Cedar repository layout including `cedar-lean-ffi` read.

Remaining:
- No Lean 4 project located that states a mechanical refinement from a protocol-level model (Veil/Leslie/lean-tla) to Rust/C code. Telltale claims a Rust<->Lean correspondence over its own protocol machine; its mechanism is "comparison-checked", not a refinement proof.
- No reusable network-semantics library with a published semantic account of Lean `IO` sockets (LeanIn names axioms instead; S31).
- Signal Shot (CSLib/Boole) results: not found.
- LeanMachines papers (IFM/ACM): not readable here (lead L1).
- Full contents of verified-consensus, Velvet and cedar-spec were not read beyond counts; dossiers cover them.
- `#print axioms` and building were not run (disk limit).
- Commit-pattern evidence for AI-generation was not collectable with shallow clones.

## 6. Open questions

1. Is the Veil temporal layer (`prove_temporal_by`) used end-to-end on a message-passing consensus protocol with fairness hypotheses discharged, or only on the examples in `Examples/Liveness`? (S19)
2. What is the exact relationship between Leslie's refinement framework and Veil's transition-system semantics; is there a translation? (S4, S19)
3. Does Telltale's "theorem-defined Rust<->Lean correspondence" cover message-passing between machines or only a single runtime slice (`ConcreteScheduledStep`)? (S15)
4. Which of the large agent-generated Lean developments (S22, S23, S25, S26) have an independent statement-fidelity audit?
5. Does any project connect an Aeneas-extracted Rust function to a Veil/Leslie/lean-tla `Next` relation? raft-lean-squad names function-level discharge as future work. (S22, S36)
6. How are Lean `Task`/`Promise` semantics specified for reasoning, if at all, beyond LeanIn's nine axioms? (S31, S32)
7. Will CSLib add concurrency or Raft content, and what is the status of its Boole pillar? (S11, S12)
8. Which Lean 3 temporal-logic or Unit-B libraries have been ported to Lean 4, if any? Zulip reports they were not kept up to date. (S41, S45)
9. What is the proof effort (person-time) behind the Lean 4 consensus developments; only Veil's effort claim (months to years for interactive proofs) was seen in a cited introduction. (S19)
10. Are the Veil case studies re-usable as specifications for code-conformance checking (for example via Loom's simulator)? (S21)
11. How is time modeled in Lean protocol libraries beyond untimed LTL (timed automata, clocks)? None located.
12. Which of the DyLean case studies include implementation-level linkage? The paper abstract mentions none. (S39)
13. What are the soundness consequences of the `axiom` declarations in LeanMachines (`_Action_ext_ax`), rumpsteak-aura (35), LeanIn (11)? (S1, S16, S31)
14. What does "any codebase" require from a Lean route given Aeneas currently excludes unsafe and concurrent Rust and has no C/Go/Java front-end in the sources read? (S36)
15. Why do README claims and repository contents differ in Leslie (C1, C2) and tla-plus-lean-workflow (C6)?

## 7. Unverified leads (excluded from evidence; Phase 2 should ignore)

- L1. LeanMachines papers (ACM DL https://dl.acm.org/doi/abs/10.1145/3796230; HAL https://hal.science/hal-04683262): pages returned no readable text.
- L2. Loom ACM version https://dl.acm.org/doi/10.1145/3776719: blocked; the author PDF was read instead (S21).
- L3. CSLib Zulip threads ("Official Announcement", "Porting from Rocq?") and HCSS 2026 talk abstract: seen only as snippets.
- L4. Telltale crate docs (docs.rs) and DeepWiki page: seen only as snippets; README and inventory read instead.
- L5. rumpsteak-aura-choreography crate docs: snippet only.
- L6. Reservoir page for lean-verification-components: not opened; repo inspected instead.
- L7. IACR news item on Bob DyLean and PLAS 2026 page (LeanDY): snippets only; LeanDY not inspected.
- L8. Lentil agent appendix (`docs/Lentil-agent-appendix.md`): snippet only.
- L9. Cedar CNCF sandbox announcement: snippet only.
- L10. Signal Shot / Boole pillar results: no source read.
- L11. LeanDY (second Lean symbolic-protocol framework): no repository read.
- L12. Aeneas user documentation site pages beyond README and `docs/user/src/README.md`.
- L13. Lean 4.34 release notes and other Lean language-server or `Std.Sat`, `Std.Time` details: fetched but not used.
- L14. Veil `Examples/LivenessMC/*`: file names read only; contents not read (the `Examples/Liveness/TLA/Consensus.lean` example is covered in S19).
- L15. Hudon's Unit-B Lean 3 code beyond `fairness.lean` header and file listing (`refinement/` directory contents).
- L16. TLA+ trace-validation and Ivy QUIC sources: handled by dossier agents.
- L17. Any Lean 4 port of Event-B beyond LeanMachines (for example a Rodin translator mentioned as "under consideration" in the README).
- L18. DyLean paper (https://eprint.iacr.org/2026/1493): only the abstract page was retrievable; the PDF could not be downloaded, so its body claims are not used as evidence.
