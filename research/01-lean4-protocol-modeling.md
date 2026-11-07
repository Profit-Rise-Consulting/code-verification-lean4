# 01 - Lean 4 (and Lean 3 lineage) protocol modeling: evidence file (Phase 1b)

Counts (this file): [FULL] entries: 27 | [CODE] entries: 24 (11 of them also have a [FULL] paper/doc read, listed as FULL+CODE) | Unverified leads (excluded from evidence): 17 | Access date for every URL: 2026-11-07.
Counting rule: an entry is FULL if the whole document was retrieved and read for the passages quoted; CODE if the repository was cloned (`--depth 1`, blob limit 1 MB) or its files were downloaded and inspected. FULL+CODE entries are counted in both totals. Phase 1 entries that were search-snippet-only have been removed from the evidence and listed under "Unverified leads".

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
https://github.com/lean-machines-central/lean-machines | SHA 9f1f001f0a | last commit 2026-11-04 | toolchain `leanprover/lean4:v4.35.0-rc3` | licence present (LICENSE) | 50 files, 11,252 LOC (7,992 code) | sorry 10 (all in `LeanMachines/NonDet/Algebra/Convergent.lean`), axiom 4, theorem 23, native_decide 0.
- README: "dedicated to the formal modeling of stateful systems. The main objective is to support a stepwise refinement methodology inspired by the Event-B formal method"; "the framework enforces the fundamental principle of correctness-by-construction: machine states, events structures and refinement steps cannot be fully constructed without discharging the prescribed proof obligations." Status line: "Project status : alpha ... may be modified without prior warning." Also: "The framework is not directly compatible with Event-B and related implementations such as Rodin."
- Core definition (`LeanMachines/Event/Basic.lean`): `class Machine (CTX) (M) extends Inhabited M where context : CTX; invariant : M -> Prop`, doc: "an `invariant` property defining the overall safety requirements of the machine states". Events are `structure _Event ... where guard (m : M) (x : alpha) : Prop := True; action (m : M) (x : alpha) (grd : guard m x) : (beta x M)`. Event kinds: `InitDet | InitNonDet | TransDet (status : Convergence) | TransNonDet ...` with `Convergence = Ordinary | Anticipated | Convergent`.
- Axioms: `_Action_ext_ax` in `Event/Basic.lean` carries the author comment "XXX : does this axiom breaks something ? (I don't think it's provable because of HEq)"; `AnticipatedEvent.ext` and `ConvergentEvent.ext` in `Event/Convergent.lean` are the other axioms.
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
- README table (read at the same SHA): "Paxos | Quorum intersection, 14 actions | Spec complete, 2 sorry" and "VRViewChange | Viewstamped Replication view change safety | 1 sorry". The code scan found no `sorry` in the Paxos example files and 2 in `LastVoting.lean`; this is recorded under Conflicts (C1). Dossier: `references/leslie`.
- README states the library covers "round-based" protocols and has `Leslie/Prob/` probabilistic examples. `docs/rust-verification.md` and `docs/rust-verification-plan.md` describe a Lean-model-plus-Verus approach for Rust code; the Rust crate listed in README (`Leslie/Rust/...`) was not present in the cloned tree (no `CoreSemantics.lean` found). This is a plan document at this SHA (see C2).
- AI-generated flag: repo contains `AGENTS.md`/`CLAUDE.md`; basis is file presence. See dossier.

**S5. lean-tla (TLA+ DSL in Lean, with vendored Leslie) [P][CODE] tags: H1, H2, D**
https://github.com/yihuang/lean-tla | SHA 99d8be4103 | last commit 2026-08-17 | toolchain `leanprover/lean4:v4.33.0-rc2` | no LICENSE file found at root | 323 files, 141,774 LOC (104,397 code) | theorem 3,404 | sorry 5 (all in `docs/research/vendor/leslie`, the vendored Leslie copy; none in the repository's own `TlaDsl`/`Bft` code) | native_decide 7 | partial 50.
- Contains `TlaDsl/` (typed DSL with `Tla.Entails`, `Tla.leadsTo`, `Tla.statePred`, tactics `tla_drop_simpa` etc.), examples `Paxos.lean`, `PaxosConsensus.lean`, `StreamletByzLiveness.lean`, ticket-lock and mutex examples. The repo includes guidance files for AI agents (CLAUDE.md/AGENTS.md style), so it is flagged as AI-assisted by file evidence.
- Verbatim liveness statement (`TlaDsl/Examples/StreamletByzLiveness.lean:1293`):
  ```lean
  theorem liveness_spec_byz (n : Nat) (Byz : Finset Node) (L : Epoch -> Node)
      (hByz : ByzOk n Byz) (e0 : Epoch) (he0 : 0 < e0) :
      Tla.Entails (HByz n Byz L)
        (Tla.leadsTo (Tla.statePred (fun s => s.cur = e0))
          (Tla.statePred (fun s => FinalSomeByz n s))) := by
  ```
  The conclusion is a leads-to under the hypotheses `HByz` (a Byzantine-fault-tolerant Streamlet spec parameterised by a leader schedule `L` and fault set `Byz` satisfying `ByzOk n Byz`); the schedule and timing assumptions are inputs to the theorem.
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

**S10. Zero-to-QED, chapter 22 "State Machine Verification" and repo [S][FULL+CODE] tags: H3, D**
https://sdiehl.github.io/zero-to-qed/22_model_checking.html and https://github.com/sdiehl/zero-to-qed | SHA cf59ccfb80 | 2026-11-01 | toolchain v4.34.1 | LICENSE present | 50 files, 9,045 LOC | theorem 405 | sorry 3 (tutorial files `Proving.lean`, `AlgebraicStructures.lean`, `Tactics.lean`) | axiom 2 (`Basics.lean`) | native_decide 7 | partial 5. Tutorial book (secondary).
- Chapter models a circuit breaker as a Lean state machine; a Rust `step` function is checked against it on "83,300 bounded single-step cases" and "2,025 trace tests". Quotes: "The tests provide evidence of correspondence, not a proof for all inputs"; limitations listed include "manual transcription", "Lean naturals are unbounded, whereas Rust integers are fixed-width", and "whether they capture the intended service behavior remains a specification question."

### 1C. Process calculi, labelled transition systems, impossibility results

**S11. CSLib repository [P][CODE] tags: H1, H2**
https://github.com/leanprover/cslib | SHA 76b7943d0b | 2026-11-05 | toolchain `leanprover/lean4:v4.36.0-rc2` | LICENSE present | 341 files, 52,113 LOC (30,796 code) | theorem 3,050 | sorry 0 | axiom 0 | native_decide 0 | opaque 2.
- Directory inventory relevant to protocols: `Cslib/Foundations/Semantics` (LTS), `Cslib/Languages/CCS`, `Cslib/Languages/Mech` (including a choreography area), `Cslib/Logics/HML`, `Cslib/Logics/Modal`, `Cslib/Computability/Distributed` (subfolders `FLP` and `Consistency`), `Cslib/Crypto/Protocols`, `Cslib/Algorithms/StatefulProcesses`. A search of the tree for "raft" returned no Raft development (only unrelated word matches).
- `Cslib/Computability/Distributed/FLP/README.md` (author Ching-Tsun Chou, "Copyright (c) 2026", Apache 2.0): "This directory contains a formalization of Volzer's proof [Volzer2004] of the famous result in distributed computing, first proved by Fischer, Lynch and Paterson [FLP1985], that asynchronous distributed consensus is impossible in the presence of even a single crash fault." Eight files (`Algorithm`, `Consensus`, `FairScheduler`, `CanReachVia`, `PseudoConsensus`, `OnePseudoConsensus`, `Impossibility`, `ZeroConsensus`). `ZeroConsensus.lean` is "included to show that the notion of an algorithm ... is not vacuous". Final theorems in `Impossibility.lean`: `OnePseudoConsensus.not_terminating`, `Consensus.one_not_exists [Fintype P] (hc : 2 <= card P)`, `Consensus.ge_one_not_exists ... (hf : f >= 1)`. This is a proof about the class of all algorithms in the model, not about one protocol instance.
- `Computability/Distributed/Consistency/Specification.lean` formalizes "several notions of consistency for replicated data types defined in [Burckhardt2014]" ordered by strength.
- Dossier: none specific; see also S12-S14.

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

**S16. rumpsteak-aura (Rust choreographic MPST with Lean directory) [P][CODE] tags: H1, H2, H3**
https://github.com/hxrts/rumpsteak-aura | SHA 024b769afe | last commit 2026-01-30 | LICENSE present | 221 Lean files, 58,185 LOC (41,738 code) | theorem 2,287 | axiom 35 (Effects/Deployment/Part2b.lean 9, Effects/DeadlockFreedom.lean 7, Effects/Preservation.lean 6, Effects/Deployment/Part1.lean 6, others) | native_decide 2 | partial 5 | sorry 0.
- README: "a Rust framework for safely and efficiently implementing message-passing asynchronous programs. It uses multiparty session types to statically guarantee the absence of communication errors such as deadlocks"; session-type core "forked from Zak Cutner's Rumpsteak". The Lean `Effects/DeadlockFreedom.lean` file declares 7 axioms, i.e. its deadlock-freedom statement rests on stated assumptions in Lean. Same author as S15 (hxrts); the relationship between the two repositories was not documented in the files read.

**S17. "Session Type State Spaces Form Lattices" (EPTCS 453) [P][FULL] tags: H1**
https://arxiv.org/pdf/2609.34927. Abstract-level facts from the text: "Mechanised in Lean 4 with two independently developed tool implementations"; "in Lean 4 (17 modules, zero sorry)"; classification of 93 distributive and 15 non-distributive cases. A structural result about session-type state spaces; no network or implementation link.

**S18. Asynchronous multiparty session types, mechanised in Rocq (ECOOP 2025) [P][FULL] non-Lean comparator; tags: D**
https://drops.dagstuhl.de/storage/00lipics/lipics-vol333-ecoop2025/LIPIcs.ECOOP.2025.31/LIPIcs.ECOOP.2025.31.pdf. Included only as comparator for what mechanised MPST covers outside Lean. Not used for Lean claims.

### 1E. Distributed protocol and consensus developments in Lean 4

**S19. Veil (framework for transition systems; distributed-protocol DSL) [P][FULL+CODE] tags: H1, H2, H3, D**
https://github.com/verse-lab/veil | SHA 57447722e7 | 2026-11-07 | toolchain v4.35.0 | LICENSE present | 495 files, 89,474 LOC (56,556 code) | theorem 560 | sorry 9 (test/regression files, e.g. `VeilTest/Regression/TemporalBasic.lean` 2, `VerifierInteractiveRollback.lean` 2, `VeilTest/WarnTrustingSmtSolver.lean` 1) | admit 6 | native_decide 24 | implemented_by 8 | partial 61 | opaque 1. Dossier: `references/Veil-Lentil-Velvet-Loom`.
- Current README (read in full): "Veil is a framework for (1) specifying, (2) implementing, (3) model-checking, and (4) proving safety and liveness properties of state transition systems, with a focus on distributed protocols." Example list includes `Examples/Liveness/` ("Temporal properties proved with `prove_temporal_by`") and `Examples/LivenessMC/` ("model checked with `#temporal_check`, including TLA+ specifications and CAVA benchmarks").
- `docs/Distributed-DSL.md` (read): "For message-passing protocols there is a more compact way: describe the program each node runs, and the network that connects the nodes." Network options (verbatim table): `synchrony` async | sync; `loss` reliable | lossy; `channels` set | multiset; `order` unordered | fifo; `faults` none | `crash f` | `byzantine p`. "The compiled module is strict by construction: a node can only read and write its own variables, and can only learn about other nodes through messages."
- `docs/DSL-Reference.md` (read): temporal properties are "LTL formulas over the runs of the system" with `temporal [electsLeader] (forall n next, WF (send n next)) /\ ... -> diamond (exists l, leader l)`; `#temporal_check` "explores every infinite run of the instance, including runs that never take an enabled action, so liveness properties usually need fairness assumptions"; `prove_temporal_by` "proves a temporal property for all instantiations of the module, in the proof mode of Lentil ... It assumes the invariants (`hInv`) rather than proving them". Veil's temporal layer uses Lentil's syntax and rules (e.g. WF1). A liveness proof example is named `Examples/Liveness/TLA/Consensus.lean`. `Examples/Demo/FloodSet/FloodSetLiveness.lean` was downloaded: 128 lines, properties `terminates`, `decisionStable`, `everyoneDecides`, with `prove_temporal_by [decisionStable]` at line 102.
- CAV 2025 paper (https://link.springer.com/content/pdf/10.1007/978-3-031-98682-6_2.pdf, read in full text): abstract: "an open-source framework for automated and interactive verification of transition systems, aimed specifically at conducting machine-assisted proofs about concurrent and distributed algorithms"; "verification conditions in first-order logic, to be discharged automatically via a range of SMT solvers". RQ1 used "16 case studies": 9 from IvyBench, 2 from Padon et al. on Paxos, 5 others; "The case studies in our set total 1704 non-empty, non-comment lines of Lean code". "Veil verifies all but 2 benchmarks in under 15 s (87.5%), and all but 4 benchmarks in under 10 s (75%)." Soundness theorem: "forall s post, tr post s <-> (forall s r, tr s r s => post r s) ... for all actions tr with no failing assertions." Environment: Lean 4.16.0, cvc5 1.2.1, Z3 4.14.0. The CAV paper text contains no discussion of liveness proofs (liveness appears only in a cited title); the later README adds them (see C3).

**S20. Veil, Loom and Dafny-style verifiers (Dafny 2026 workshop paper) [P][FULL] tags: H1, H2**
https://verse-lab.org/papers/veil-dafny26.pdf. Describes Veil's design and a Ring leader-election example with both automated and interactive proof (`theorem send_tr_single_leader'`); states external SMT solvers "do not have to be trusted: Veil supports proof reconstruction (at a 3-5x performance penalty)". No occurrences of "liveness" or "temporal" in the retrieved text.

**S21. Loom (POPL 2026) paper and repository [P][FULL+CODE] tags: H1, H2, H3**
https://verse-lab.org/papers/loom-popl26.pdf and https://github.com/verse-lab/loom | SHA 78928abc90 | 2026-02-21 | toolchain v4.24.0 | LICENSE present | 38 files, 9,628 LOC (7,068 code) | theorem 228 | partial 31 | axiom 1 (`Loom/SMT.lean:215`: `axiom trust_smt : forall (p : Prop), p`, used to close goals delegated to an SMT solver) | admit 1.
- Paper: "Loom: A Framework for Automated Generation of Foundational Multi-Modal Verifiers" (README title). Abstract mentions verifiers for "(1) distributed protocol safety and (2) Dafny-style specification" code; the Veil 2.0 benchmark suite "contains 17 specifications of 15 different distributed protocols". Section 7 "Combining Runtime and Deductive Verification of Distributed Protocols" describes a simulator for protocols "implemented using Lean's ..." (execution inside Lean, i.e. runtime checking of the model, not extraction to another language). Loom is built on "monadic shallow embedding of an executable program semantics into Lean 4", "Monad Transformer Algebras" for weakest preconditions. Case studies directory: Cashmere, Velvet.
- Velvet (verse-lab/velvet) counts only, SHA b2a349a570, 2026-11-05, toolchain v4.35.0: 140 files, 34,674 LOC, theorem 719, sorry 0, native_decide 22, partial 5.

**S22. raft-lean-squad (Rust raft-rs with Lean model) [P][FULL+CODE] tags: H2, H3, D**
https://github.com/dsyme/raft-lean-squad | SHA ab7a184fbd | last commit 2026-06-03 | LICENSE present | 80 Lean files, 21,355 LOC (8,726 code) | theorem 779 (token count; REPORT says 716, see C4) | sorry 0 | axiom 0 | native_decide 2. `formal-verification/lean/lean-toolchain` says v4.28.0 while the REPORT cites 4.30.0-rc2 (C4). Dossier: `references/raft-lean-squad`.
- REPORT quotes (fetched earlier in full, rechecked metadata): "716 theorems, 79 Lean files, 0 sorry"; models "not mechanically extracted from Rust source"; abstract u64 modeled as `Nat`; no "full message-passing network model", no liveness, no concurrency/IO. Correspondence checked by `#guard` assertions; "concrete function-level discharge" proposed as future work. The repo is produced by automated agents (".github/workflows/lean-squad.md", "agentic-workflows" files): flagged AI-generated on file evidence. Repository also contains `Charon.toml` (Aeneas front-end config) but the Lean models are hand-written per the REPORT.

**S23. ConsensusVerifier ("The End of Consensus Research?" / AI-greement) [P][FULL] tags: H2, H3, D**
https://arxiv.org/html/2610.30114 (read earlier in full; no repository URL was found in the paper text). About 135K lines of Lean covering Multi-Paxos, FastPaxos, SwiftPaxos, EPaxos*, Bodega and A-greement; proves linearizability of protocol models; quote: "our Lean proofs cover a protocol model whose correspondence to the prototype is additionally checked by a human consensus expert"; "currently targets safety only". The authors distinguish formalizing from pseudocode versus implementation code; whether to formalize implementation behavior directly "remains an open question". Agent-generated proofs; kernel-checked.

**S24. verified-consensus (Lean 4, decoupled consensus) [P][CODE, counts only] tags: H2**
https://github.com/fradamt/verified-consensus | SHA d379754762 | 2026-10-12 | toolchain v4.34.1 | LICENSE present | 1,047 files, 334,092 LOC (256,931 code) | theorem 8,030 | sorry 0 | axiom 0 | native_decide 5 | partial 1. Content not read beyond counts. Dossier: `references/verified-consensus`.

**S25. lean-dag (Mysticeti-style DAG consensus) [P][CODE] tags: H2, D**
https://github.com/mlegner/lean-dag | SHA 243db27917 | 2026-09-07 | toolchain v4.32.2 | LICENSE present | 478 files, 78,475 LOC (43,782 code) | theorem 2,379 | sorry 0 | axiom 0.
- README provenance (verbatim): "Code and prose in this project were co-written with heavy LLM assistance. The Lean proofs are machine-checked ... but whether the definitions and theorem statements capture their intended meaning, ... has only human-plus-LLM review behind it. Read critically." Proved: safety "with no network assumption of any kind"; liveness "above eventual DAG synchrony, a structural condition on the DAG under which no liveness theorem mentions time"; "view convergence - after stabilisation, whatever one correct validator holds reaches every correct validator within Delta". Committee sizes `n >= 3f+1` (core), `n >= 5f+1`, `n = 3f + 2p - 1`, `n >= 5 fb + 3 fc + 1`, `n >= 2f+1` for crash only.

**S26. lean-verification-components / leanbook (Raft, Paxos, PBFT files) [P][CODE] tags: H2, D**
https://github.com/solanaqubits/leanbook | SHA 5276529c6c | 2026-11-05 | toolchain v4.33.1 | LICENSE present. Contains `Verification/DistributedRaft*.lean` (about 20 files), `DistributedPaxos*`, `DistributedPBFTConsensus`, `BFTConsensusQuorum`. In `DistributedRaftConsensus.lean`: `theorem election_safety_unique_leader ... (hA_votes : forall x in A, votedFor x = c1) (hB_votes : forall x in B, votedFor x = c2) : c1 = c2` is proved from majority-quorum overlap (`majority_intersection_nonempty`), with the repo's own comment on `leader_commit_quorum_overlap`: "Set overlap only, without a log-freshness rule or a fault model." The docs describe an automated verification hierarchy ("alpha/system, beta/full checkpoint, gamma/direction ...") in which "Proof authors can propose updates" to specs, i.e. an automated-agent process; flagged likely AI-generated on that basis. Reservoir indexing was not checked.

**S27. tla-plus-lean-workflow (STTT submission artifact) [P][CODE] tags: H3, D**
https://github.com/akkeshavan/tla-plus-lean-workflow | SHA 9910342183 | 2026-06-29 | no toolchain file at root | no licence file | 20 Lean files, 3,377 LOC | theorem 69 | sorry 0 in the token scan, but source comments say "All proof obligations are marked `sorry` pending Veil/SMT discharge" (`VerificationContinuum/OpenRaft/Propositional.lean:10`) and "two sorry stubs" (`VeilProofs/PaxosVeil.lean`, `OpenRaftVeil.lean`), so the scan and comments disagree (the stubs are described in comments or Veil-side files not matched by the token scan). Pipeline: "Phase 1 TLA+ model checking; Phase 2 Lean 4 functional model; Phase 3 Lean 4 propositional proofs (inductive invariants via Veil/SMT); Phase 4 Rust implementation (Aeneas functional translation where applicable); Phase 5 Solarkraft runtime monitoring". Six case studies (two-phase commit, token mutex, Raft election, openraft, Paxos, Lightning). Author: single independent researcher.

**S28. compositional-temporal-safety [P][CODE] tags: H2**
https://github.com/repowazdogz-droid/compositional-temporal-safety | SHA 7a812bdb46 | 2026-07-20 | LICENSE present | 7 Lean files, 481 LOC (267 code) | theorem 19 | sorry 0. README: assume-guarantee composition of per-agent invariants for a roster of agents sharing a budget and a critical-section token; proves `totalSpent rs s.spent <= cap` and mutual exclusion "at every reachable state" for "an arbitrary finite Nodup roster". Claims are mapped to "evidence grades" in the repo. The README says the artefact is a verification artefact for a toy multi-agent system and names unclosed gaps shared with sibling repos.

### 1F. Concurrency models, separation logic, Lean runtime

**S29. Iris-Lean (Iris in Lean 4) [P][CODE] tags: H1, H2**
https://github.com/leanprover-community/iris-lean | SHA 8293092f09 | 2026-11-05 | LICENSE present | 324 files, 100,399 LOC (74,181 code) | theorem 5,454 | axiom 5 (all in `Iris/IrisTest/Benchmarking/BenchmarkHeapLangList.lean`: `isList_nil`, `isList_cons`, `nil_spec`, `cons_spec`, `length_spec`) | sorry 0 | opaque 3 | partial 46. HeapLang is under `Iris/IrisTest/HeapLang/` (WeakestPre, HeapTactics, Apply, Notation).

**S30. "Iris in Lean" paper (arXiv 2609.24252) [P][FULL] tags: H1, H2**
https://arxiv.org/pdf/2609.24252. Describes the Lean port of Iris (ghost resources, step-indexed BI, proof mode) and a port of HeapLang (section 2.2). Quote: "While Iris has mostly been used to reason about safety properties, several projects have also investigated applications to ... security properties such as non-interference, as well as probabilistic properties and liveness." The port paper records limitations of the proof-mode rewriting ("does not yet support rewriting by single entailments"). It does not present a distributed protocol verification.

**S31. LeanIn (concurrency runtime with a bridge from model to runtime) [P][CODE] tags: H1, H3, D**
https://github.com/scidonia/leanin | SHA 3015043d80 | 2026-10-31 | toolchain v4.35.0 | LICENSE present | 78 files, 21,376 LOC (13,183 code) | theorem 272 | axiom 11 (`LeanIn/Theory/Bridge.lean` 9, `LeanIn/Test/BridgeControls.lean` 2) | opaque 3 | partial 24 | extern 20 | sorry 0.
- README: "a concurrency runtime with a work-stealing scheduler for Lean 4 ... on par with other highly concurrent languages but which lives in a proof assistant." Benchmark table compares it with Node 22 and the shipped `Std.Http.Server` (for example "depth 1 | shipped Std.Http.Server | 5 323 | 5 584 | 7 626" requests/s against leanin "62 568 | 147 791 | 126 481").
- `Bridge.lean` header (verbatim): "The bridge: the only axioms. `World.lean` proves the model's properties - mutual exclusion, no notification memory, permitted spurious wakeups, no fairness. None of that is assumed. What is assumed is the connection between that model and the running program ... Lean gives `IO`/`BaseIO` no semantics (`docs/proof-strategy.md` Gap A), so the statement 'running this operation advances the world like that' cannot be derived - it has to be named." Nine operation contracts (mutex create/lock/unlock, condvar create/wait/notify, clock) form the TCB.
- Companion document `docs/lean-concurrency-today.md` (read in full): table of Lean's concurrency surface (Task, Promise, `Std.Async`, `Selector`, timers, TCP/UDP, HTTP server); gap table row "proofs | none, anywhere | build (the actual research)"; `race` "does not cancel the loser".

**S32. Lean reference manual: Logical Model of IO, Tasks and Threads, Asynchronous Programming [P][FULL] tags: H1, H3**
https://lean-lang.org/doc/reference/latest/IO/Logical-Model/ and https://lean-lang.org/doc/reference/latest/IO/Async/. Logical-model page: "Side effects, which must be executed in the correct order, are abstractly described in Lean's logic. When programs are run, the Lean runtime system is responsible for actually carrying out the described effects." `IO alpha` "can be thought of as a state monad in which the state is the entire world." Async page: "The Async monad provides tools and abstractions for constructing asynchronous programs that can safely multiplex different sources of data. Typical use cases include network servers and other interactive applications." Neither page gives a formal semantics of concurrent interleaving or a memory model.

**S33. Std.Http in Lean core (HTTP/1.1 protocol state machine) [P][CODE] tags: H1, H3**
https://github.com/leanprover/lean4/tree/master/src/Std/Http (files downloaded individually from master; master SHA a2f1a642ab at 2026-11-06 per the commits API; not a clone). 48 files, 13,921 LOC (6,327 code) | theorem 24 | partial 21 | sorry 0. `Protocol/H1.lean` header: "HTTP/1.1 Protocol State Machine ... implements the core HTTP/1.1 protocol state machine that handles parsing requests/responses and generating output. The machine is direction-aware, supporting both server mode ... and client mode". The 24 theorems are in data-helper files (`Data/URI/Encoding.lean` 13, `Internal/LowerCase.lean` 5, `Internal/Char.lean` 2, `Internal/MultiMap.lean` 2, others); no theorem files exist under `Protocol/`. Authors: Sofia Rodrigues (Lean FRO). Dossier: `references/Std.Http`.

### 1G. Linking models to implementations

**S34. Cedar paper "How We Built Cedar: A Verification-Guided Approach" [P][FULL] tags: H3, D**
https://arxiv.org/pdf/2407.01688. Describes "verification-guided development" (VGD): Lean models written to be "used as oracles for differential testing" and "differential random testing [29] to increase our confidence that the behavior of our formal models matches that of our production" Rust code. Properties proved in Lean include "Forbid trumps permit". The model-to-code link is by differential random testing, per the paper's own wording.

**S35. cedar-spec repository [P][CODE counts] tags: H3**
https://github.com/cedar-policy/cedar-spec | SHA a4c5a89cd1 | 2026-11-06 | toolchain v4.35.0 | LICENSE present | 388 files, 126,845 LOC (102,084 code) | theorem 2,600 | sorry 0 | axiom 0 | native_decide 26 | implemented_by 2 | opaque 2 | partial 42. FLoC 2026 slides by de Moura (https://leodemoura.github.io/static/floc26/, read): "Executable Lean model alongside Rust production code. Lean model ~10x smaller than Rust"; "~100M differential random tests nightly. Lean: 5 us/test. Rust: 7 us/test"; "Release gate: no Cedar version ships unless model, proofs, and differential tests are current." AWS blog (https://aws.amazon.com/blogs/opensource/lean-into-verified-software-development/, read): "Over the past 6 months, we built a formal model of Cedar in Lean, and proved that Cedar's components satisfy key safety and security properties." [S] for the blog. Cedar is an authorization-policy evaluator, not a distributed protocol. Dossier: `references/Cedar`.

**S36. Aeneas (Rust to Lean functional translation) [P][CODE+FULL] tags: H3, H4**
https://github.com/AeneasVerif/aeneas | SHA c5eb4a28ce | 2026-11-07 | LICENSE.md present. README lines 124 and 138-139 (read): Aeneas does not "support unsafe code and concurrent code"; unsupported features listed: "unsafe code", "concurrency". `docs/user/src/README.md:16`: "no concurrent execution: long-term effort. We plan to address coarse-grained parallelism as a long-term goal." Aeneas project page (https://aeneasverif.github.io/projects/): Charon "acts as an interface between the rustc compiler and program verification projects ... operates on MIR ... converts MIR code to ULLBC", with JSON serializers. Aeneas paper (https://arxiv.org/pdf/2206.07185, ICFP 2022) abstract: "we tweak our semantics to abstract the aliasing graph in the presence of function calls ... backward functions, which obviates the need for prophecy variables"; backends in the paper: F* and a Coq backend in the works (the paper predates the Lean backend). Dossier: `references/Aeneas-SymCrypt`.

**S37. SymCrypt-in-Rust verification with Aeneas (arXiv 2609.15648, draft report) [P][FULL] tags: H3**
https://arxiv.org/pdf/2609.15648. Authors Ho, Fournet, Protzenko, Naehrig and others; "We target production code written in Rust ... Rust's ownership discipline enables Aeneas to extract a pure model of this code in Lean". Cryptographic primitives, sequential code; no protocol-level model. Dossier: `references/Aeneas-SymCrypt`.

**S38. Federated verification architecture (Bullish) (arXiv 2606.02019) [P][FULL] tags: H3, D**
https://arxiv.org/pdf/2606.02019. "We propose a federated architecture for production formal verification" with TLA+ obligations discharged "by citing an equivalent theorem in a structurally distinct kernel" (Coq `Print Assumptions`, Lean 4 `#print axioms`); "agreement is build-system-enforced rather than a logical soundness theorem; the residual correspondence-certificate question is named as an open problem in section 6.3". Financial-systems case; Lean is one of several kernels.

### 1H. Security protocols (symbolic model)

**S39. DyLean repository and Bob DyLean paper [P][FULL+CODE] tags: H1, H2, H3, D**
https://github.com/BobDyLean/dylean | SHA 1662f9ad82 | 2026-11-06 | toolchain v4.36.0-rc1 | LICENSE present | 98 files, 26,319 LOC (21,464 code) | theorem 597 | sorry 34 (`Tests/Meta/Step.lean` 29, `DY/Meta/LetUtils.lean` 5; may be meta-code that emits `sorry`) | admit 1 | native_decide 14 | opaque 1. Paper-artifact repo https://github.com/BobDyLean/bobdylean-paper-artifact SHA e24fe13d68, 2026-07-17, toolchain v4.32.0, 95 files, 26,167 LOC. Paper (https://eprint.iacr.org/2026/1493, Wallez and Cremers, published 2026-07-21, not peer reviewed per the source's trust note): "DyLean, a framework for the symbolic analysis of cryptographic protocols in the Lean theorem prover ... comprises both a flexible general-purpose symbolic semantics, as well as a concrete proof methodology"; "based on trace invariants"; case studies "merkle trees, ratcheting protocols, post-quantum protocols, and protocols analyzed under different equational theories". README disclaimer: "We are actively working on DyLean, expect breaking changes." Example directories: `ACME`, `MerkleTree`, `Ratchet`, `SignedDH`, `SignedDHKEM`. Symbolic model: attacker may only use cryptographic functions as black boxes (Wallez talk slides S40).

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
| S1 | lean-machines | 9f1f001f0a | v4.35.0-rc3 | 2026-11-04 | 11,252 | 23 | 10 | 4 | 0 | 0 | 0 | 0 | 0 |
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
| S19 | veil | 57447722e7 | v4.35.0 | 2026-11-07 | 89,474 | 560 | 9 | 0 | 24 | 8 | n/r | 1 | 61 |
| S21 | loom | 78928abc90 | v4.24.0 | 2026-02-21 | 9,628 | 228 | 0 | 1 | 0 | 0 | n/r | 0 | 31 |
| S21 | velvet | b2a349a570 | v4.35.0 | 2026-11-05 | 34,674 | 719 | 0 | 0 | 22 | 0 | n/r | 0 | 5 |
| S22 | raft-lean-squad | ab7a184fbd | v4.28.0 (REPORT: 4.30.0-rc2) | 2026-06-03 | 21,355 | 779 | 0 | 0 | 2 | 0 | n/r | 0 | 0 |
| S24 | verified-consensus | d379754762 | v4.34.1 | 2026-10-12 | 334,092 | 8,030 | 0 | 0 | 5 | 0 | n/r | 0 | 1 |
| S25 | lean-dag | 243db27917 | v4.32.2 | 2026-09-07 | 78,475 | 2,379 | 0 | 0 | 0 | 0 | n/r | 0 | 0 |
| S27 | tla-plus-lean-workflow | 9910342183 | n/a | 2026-06-29 | 3,377 | 69 | 0 (comments mention sorry stubs) | 0 | 0 | 0 | n/r | 0 | 0 |
| S28 | compositional-temporal-safety | 7a812bdb46 | n/a | 2026-07-20 | 481 | 19 | 0 | 0 | 0 | 0 | n/r | 0 | 0 |
| S29 | iris-lean | 8293092f09 | n/r (in Iris/) | 2026-11-05 | 100,399 | 5,454 | 0 | 5 (benchmark file) | 0 | 0 | n/r | 3 | 46 |
| S31 | leanin | 3015043d80 | v4.35.0 | 2026-10-31 | 21,376 | 272 | 0 | 11 | 0 | 0 | 20 | 3 | 24 |
| S35 | cedar-spec | a4c5a89cd1 | v4.35.0 | 2026-11-06 | 126,845 | 2,600 | 0 | 0 | 26 | 2 | n/r | 2 | 42 |
| S39 | dylean | 1662f9ad82 | v4.36.0-rc1 | 2026-11-06 | 26,319 | 597 | 34 | 0 | 14 | 0 | n/r | 1 | 0 |
| S33 | lean4 src/Std/Http | a2f1a642ab (master) | n/a | 2026-11-06 | 13,921 | 24 | 0 | 0 | 0 | 0 | n/r | 0 | 21 |
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
| Leslie's Rust verification doc is a plan, and the Rust crate was not in the tree (S4, C2). | lean-protocol-support (Lean 3) targeted network programs through a monad interface (S43). |

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

- C1. Leslie `sorry` counts. README table: "Paxos ... Spec complete, 2 sorry", "VRViewChange ... 1 sorry". Code scan at SHA 58e58b2502: 5 `sorry` in total, located in `Refinement.lean` (2), `Examples/LastVoting.lean` (2) and `Prob/Spike/ASTSanity.lean` (1); none in Paxos files. Not reconciled (README may be stale, or the counting may differ).
- C2. Leslie Rust verification. README refers to a `Leslie/Rust/` area and a `v2` document; the cloned tree contains `docs/rust-verification.md` and `docs/rust-verification-plan.md` but no `Leslie/Rust/CoreSemantics.lean`. The docs describe a Lean-model-plus-Verus plan; whether implementation exists elsewhere is unknown.
- C3. Veil liveness. Earlier materials (Phase 1 reading of the README at that time, the CAV 2025 and Dafny 2026 paper texts) describe safety verification and have no liveness section; the README at SHA 57447722e7 says Veil handles "safety and liveness properties" and documents `#temporal_check` and `prove_temporal_by`. Both are recorded; the later README is more recent. The Phase 1 version of this file recorded Veil as "safety-only" and is superseded by this entry.
- C4. raft-lean-squad numbers and toolchain. REPORT: "716 theorems, 79 Lean files", Lean 4.30.0-rc2; CORRESPONDENCE document (run-dependent) uses a different count (681 in the version read); repo scan: 80 files and 779 `theorem`/`lemma` tokens; `lean-toolchain` v4.28.0. Differences may be due to document staleness or counting method.
- C5. CSLib and Raft. A secondary source seen in Phase 1 claimed a Raft port in CSLib; no Raft content exists in the cloned CSLib tree (SHA 76b7943d0b). Recorded as unconfirmed; the claim is removed from evidence.
- C6. tla-plus-lean-workflow `sorry`. The token scan finds 0 `sorry`, while source comments say proof obligations are "marked `sorry` pending Veil/SMT discharge" and mention "two sorry stubs". Not reconciled.
- C7. Telltale vs rumpsteak-aura. Same author; Telltale scan has 0 axioms in 830 files, rumpsteak-aura has 35 axioms in 221 files. The relationship (rename, rewrite, separate lines) is not stated in the files read.
- C8. Iris-Lean location of HeapLang. The Iris-Lean paper presents HeapLang as part of the library (section 2.2); in the repository it sits in `Iris/IrisTest/HeapLang` (a test-named directory) and the only `axiom` declarations are in a benchmark file. Not clear whether HeapLang is considered core or demonstration.

## 5. Gaps closed and gaps remaining (against the Phase 1 report)

Closed in Phase 1b:
- Veil liveness status read from the current README/docs rather than inferred (C3).
- CSLib protocol content inspected in the tree: FLP proof and consistency specifications found; Raft port not found (C5).
- Lean 3 lineage located and inspected: unitb/temporal-logic, unitb-semantics, GaloisInc/lean-protocol-support, lean-temporal, Zulip thread, Hudon slides (S41-S46).
- Telltale and rumpsteak-aura TCB statements read.
- Lean runtime/concurrency documentation read (S31, S32).

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
- L14. Veil `Examples/Liveness/TLA/Consensus.lean` and `Examples/LivenessMC/*`: file names read from docs only; contents not read.
- L15. Hudon's Unit-B Lean 3 code beyond `fairness.lean` header and file listing (`refinement/` directory contents).
- L16. TLA+ trace-validation and Ivy QUIC sources: handled by dossier agents.
- L17. Any Lean 4 port of Event-B beyond LeanMachines (for example a Rodin translator mentioned as "under consideration" in the README).
