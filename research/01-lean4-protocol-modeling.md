# 01 - Lean 4 (and Lean 3 lineage) protocol modeling: evidence log

Phase 1: gather evidence only. This file records findings; it does not analyse, conclude or judge the hypothesis.

Hypothesis under test (from the task brief): "Lean 4 cannot verify protocols and therefore cannot check code implementations of them."

Access date for every source below: 2026-11-07. Tags: [P] = primary (the project's own repo/docs, the paper itself, official Lean docs, Zulip); [S] = secondary (third-party summary, blog, tutorial, aggregator, vendor marketing). Evidence basis: [F] = full page fetched and read; [X] = search-result excerpt only (not fully read). Quoted text is verbatim from the excerpt/page retrieved.

Layout note: in each topic, "Recorded items pointing toward 'cannot'" and "Recorded items pointing toward 'can'" are placed side by side only so that both are visible. The placement is a filing convenience, not a judgement of weight, and some items are mixed (marked "mixed").

---

## 1. Source register

| ID | Source | URL | Tag / basis |
|---|---|---|---|
| S1 | LeanMachines README | https://github.com/lean-machines-central/lean-machines/blob/main/README.md | P / X |
| S2 | LeanMachines journal paper "State-based Modeling with Refinement (a Lean4 Framework)" | https://dl.acm.org/doi/abs/10.1145/3796230 | P / X |
| S3 | LeanMachines IFM 2024 paper "Stateful Functional Modeling with Refinement (a Lean4 Framework)" | https://dl.acm.org/doi/10.1007/978-3-031-76554-4_7 | P / X |
| S4 | LeanMachines examples repo | https://github.com/lean-machines-central/lean-machines-examples | P / X |
| S5 | Veil repo README | https://github.com/verse-lab/veil | P / F |
| S6 | Veil CAV 2025 paper | https://link.springer.com/chapter/10.1007/978-3-031-98682-6_2 | P / X |
| S7 | Veil DSL reference | https://github.com/verse-lab/veil/blob/main/docs/DSL-Reference.md | P / X |
| S8 | "Lessons from Building an Auto-Active Verifier in Lean" (Dafny'26) | https://verse-lab.org/papers/veil-dafny26.pdf | P / X |
| S9 | veil.dev site | https://veil.dev/ | P / X |
| S10 | Lean use-case page for Veil | https://lean-lang.org/use-cases/veil/ | P / X |
| S11 | Loom paper "Foundational Multi-Modal Program Verifiers" (POPL 2026) | https://dl.acm.org/doi/10.1145/3776719 | P / X |
| S12 | Velvet v2.0 release notes | https://github.com/verse-lab/velvet/releases/tag/v2.0 | P / X |
| S13 | Leslie (TLA shallow embedding in Lean 4) | https://github.com/rupakm/leslie | P / F |
| S14 | lean-tla (deep embedding of TLA+) | https://github.com/yihuang/lean-tla | P / F |
| S15 | Lentil (LTL proof mode) repo | https://github.com/verse-lab/Lentil | P / X |
| S16 | Lentil reference manual | https://github.com/verse-lab/Lentil/blob/main/docs/Lentil-reference.md | P / X |
| S17 | LeanLTL, ITP 2025 (LIPIcs) | https://drops.dagstuhl.de/entities/document/10.4230/LIPIcs.ITP.2025.37 | P / X |
| S18 | LeanLTL arXiv version | https://arxiv.org/html/2507.01780v1 | P / X |
| S19 | LeanearTemporalLogic (student LTL formalization) | https://github.com/mrigankpawagi/LeanearTemporalLogic | P / X |
| S20 | CSLib repo README | https://github.com/leanprover/cslib/ | P / X |
| S21 | CSLib white paper | https://arxiv.org/html/2602.04846 | P / X |
| S22 | "Computer Science as Infrastructure: the Spine of CSLib" | https://arxiv.org/html/2602.15078 | P / X |
| S23 | Hennessy-Milner Logic in CSLib | https://arxiv.org/html/2602.15409 | P / X |
| S24 | CSLib Zulip: Official Announcement | https://leanprover-community.github.io/archive/stream/513188-CSLib/topic/Official.20Announcement.html | P / X |
| S25 | CSLib Zulip: Porting from Rocq? | https://leanprover-community.github.io/archive/stream/513188-CSLib/topic/Porting.20from.20rocq.20.3F.html | P / X |
| S26 | CSLib Pillar 2 / Boole talk abstract (HCSS 2026) | https://sos-vo.org/group/hcss_conference/2026/barrett | P / X |
| S27 | Iris-Lean repo | https://github.com/leanprover-community/iris-lean | P / X |
| S28 | "Iris in Lean" paper | https://arxiv.org/html/2609.24252v1 | P / X |
| S29 | Telltale (MPST, Lean proof stack) crate docs | https://docs.rs/crate/telltale-theory/latest | P / X |
| S30 | Telltale session types page (DeepWiki) | https://deepwiki.com/hxrts/telltale/3.2-session-types-and-coinductive-theory | S / X |
| S31 | Rumpsteak-Aura choreography crate docs | https://docs.rs/crate/rumpsteak-aura-choreography/0.9.1 | P / X |
| S32 | "Session Type State Spaces Form Lattices" | https://arxiv.org/pdf/2609.34927 | P / X |
| S33 | MPST asynchronous mechanised proof (ECOOP 2025, Rocq - non-Lean comparator) | https://drops.dagstuhl.de/storage/00lipics/lipics-vol333-ecoop2025/LIPIcs.ECOOP.2025.31/LIPIcs.ECOOP.2025.31.pdf | P / X |
| S34 | raft-lean-squad REPORT | https://github.com/dsyme/raft-lean-squad/blob/main/formal-verification/REPORT.md | P / F |
| S35 | lean-verification-components (Reservoir) | https://reservoir.lean-lang.org/@solanaqubits/lean-verification-components | P / X |
| S36 | verified-consensus (decoupled consensus, Lean 4) | https://github.com/fradamt/verified-consensus/blob/main/README.md | P / X |
| S37 | lean-dag (Mysticeti-style DAG consensus) | https://github.com/mlegner/lean-dag | P / X |
| S38 | "The End of Consensus Research?" (ConsensusVerifier / AI-greement) | https://arxiv.org/html/2610.30114 | P / F |
| S39 | tla-plus-lean-workflow | https://github.com/akkeshavan/tla-plus-lean-workflow | P / X |
| S40 | compositional-temporal-safety | https://github.com/repowazdogz-droid/compositional-temporal-safety | P / X |
| S41 | "From Zero to QED" ch. 22, State Machine Verification | https://sdiehl.github.io/zero-to-qed/22_model_checking.html | S / F |
| S42 | "How We Built Cedar: A Verification-Guided Approach" | https://ar5iv.labs.arxiv.org/html/2407.01688 | P / X |
| S43 | AWS blog "Lean Into Verified Software Development" | https://aws.amazon.com/blogs/opensource/lean-into-verified-software-development/ | P / X |
| S44 | cedar-spec repo | https://github.com/cedar-policy/cedar-spec | P / X |
| S45 | DyLean repo | https://github.com/BobDyLean/dylean | P / X |
| S46 | Bob DyLean ePrint abstract (IACR news) | https://www.iacr.org/news/item/29076 | P / X |
| S47 | Wallez talk slides "Proofs of Security Protocols in Lean" | https://www.twal.org/pdf/slides/2025_12_google.pdf | P / X |
| S48 | Aeneas user documentation | https://aeneasverif.github.io/aeneas/ | P / X |
| S49 | Aeneas paper (arXiv 2206.07185) | https://arxiv.org/html/2206.07185 | P / X |
| S50 | Aeneas projects page | https://aeneasverif.github.io/projects/ | P / X |
| S51 | Lean reference manual: Asynchronous Programming | https://lean-lang.org/doc/reference/latest/IO/Async/ | P / X |
| S52 | LeanIn (Tokio-inspired runtime with proofs) | https://github.com/scidonia/leanin/ | P / X |
| S53 | LeanIn "Lean 4 concurrency as it stands" | https://github.com/scidonia/leanin/blob/main/docs/lean-concurrency-today.md | P (third-party project doc) / X |
| S54 | Lean core: HTTP/1.1 protocol state machine source | https://github.com/leanprover/lean4/blob/b35cb10cf2ced51a2fd1880cfdbf75c71ab7a35a/src/Std/Http/Protocol/H1.lean | P / X |
| S55 | Lean 3 papers page (community) | https://leanprover-community.github.io/lean3/papers.html | P / X |
| S56 | Theorem Proving in Lean (Lean 3) introduction | https://leanprover.github.io/theorem_proving_in_lean/introduction.html | P / X |
| S57 | POPL 2017 Lean presentation | https://lean-lang.org/presentations/20170116_POPL/ | P / X |
| S58 | de Moura, "The Lean Theorem Prover: Design, Evolution, and Impact" (FLoC 2026) | https://leodemoura.github.io/static/floc26/ | P / X |
| S59 | de Moura, "Lean 4: A Unified Platform for Programming and Verification" (NFM 2026) | https://leodemoura.github.io/static/nfm2026/ | P / X |
| S60 | Lean 4.34.0 release notes | https://lean-lang.org/doc/reference/latest/releases/v4.34.0/ | P / X |
| S61 | "Federated Formal Verification" (TLA+ + Lean 4 + Coq etc.) | https://arxiv.org/html/2606.02019 | P / X |
| S62 | Lean Zulip: System programming using Lean 4 (2021) | https://leanprover-community.github.io/archive/stream/270676-lean4/topic/System.20programming.20using.20Lean.204.html | P / X |
| S63 | Lean Zulip: plans for network programming API (2021) | https://leanprover-community.github.io/archive/stream/270676-lean4/topic/plans.20for.20network.20programming.20API.html | P / X |
| S64 | Lamport, TLA+ chapter (definition of behaviors/actions) | https://lamport.azurewebsites.net/pubs/spec-book-chap.pdf | P / X (non-Lean reference) |
| S65 | Raft paper (TLA+ spec and proof of safety) | https://raft.github.io/raft.pdf | P / X (non-Lean reference) |
| S66 | awesome-formal-methods: temporal logic notes (liveness/fairness) | https://github.com/yihuang/awesome-formal-methods/blob/main/docs/01-fundamentals/temporal-logic.md | S / X |
| S67 | BaDaaS replication-algorithm-overview: Raft | https://github.com/BaDaaS/replication-algorithm-overview/blob/main/0033-raft/README.md | S / X |
| S68 | Quint "Reference of Formal Verification Tools" | https://quint.sh/guides/formal-verification-tools | S / X |
| S69 | Chronos Vault dev.to post (Lean 4 "184 theorems") | https://dev.to/chronosvault/184-theorems-zero-sorry-how-we-formally-verified-a-multi-chain-protocol-57g9 | S (vendor blog) / X |
| S70 | formal-verification-skill (TLA+ and Lean 4 agent skill) | https://github.com/philipk19238/formal-verification-skill/blob/main/SKILL.md | S / X |

Count: 70 distinct sources (of which 6 are non-Lean comparators/background: S33, S64, S65, S66, S67, S68; 7 are secondary/vendor: S30, S41, S66-S70).

---

## 2. Transition systems, state machines, event-based models

### Facts recorded

- LeanMachines: "a Lean4 framework for the modeling and refinement of stateful systems", inspired by Event-B, providing "contexts, machines, events and, most importantly, the associated refinement principles" ([S1]). Status line: "The framework is in alpha stage of development and may be modified without prior warning and without ensuring non-regression." ([S1]). Root file lists version 0.2.0 "beta" ([S1] LeanMachines.lean, via search excerpt).
- LeanMachines design claim: "machine states, events structures and refinement steps cannot be fully constructed without discharging the prescribed proof obligations" ([S1]); "The embedding is very shallow in that all the constructions are directly based on the Lean4 logic and abstractions, especially dependently-typed structures and typeclasses." ([S3]).
- LeanMachines compatibility note: "The framework is not directly compatible with Event-B and related implementations such as Rodin (although a translator is under consideration)." ([S1]).
- LeanMachines examples: Buffer, Event-B Bridge and Courses, VDM AlarmSystem ([S4]). The retrieved excerpts did not show a distributed or message-passing protocol example.
- Veil: "a *foundational* framework for (1) specifying, (2) implementing, (3) testing, and (4) proving safety (and, in the future, liveness) properties of state transition systems, with a focus on distributed protocols" ([S5]). Modules declare state, initial state, procedures, actions and transitions; "Actions are **atomic**: they execute either fully or not at all" ([S7]).
- CSLib provides a generic LTS structure: `structure LTS (State : Type u) (Label : Type v) where Tr : State → Label → State → Prop` (quoted in a third-party summary; the paper text states "Thus, an LTS State Label consists of a transition relation Tr" in [S23] and "We similarly bundle both inside structures" in [S22]).
- Lean core itself contains a protocol state machine: "HTTP/1.1 Protocol State Machine ... The machine is direction-aware, supporting both server mode (receiving requests) and client mode (receiving responses)." ([S54]). The excerpt does not say whether proofs about it exist.
- Lentil defines executions as infinite state sequences: "`exec σ` | `Nat → σ` | An execution: the state at each time index. Executions are infinite." and `action σ : σ → σ → Prop` ([S16]).
- Lamport's definition for context: "A behavior is an infinite sequence of states; the steps of a behavior are its successive pairs of states." ([S64]).

| Recorded items pointing toward "cannot" | Recorded items pointing toward "can" |
|---|---|
| LeanMachines labelled "alpha" and "may be modified without prior warning" ([S1]). | LeanMachines enforces proof obligations for machines/events/refinement as typeclass constructions ([S1], [S3]). |
| Veil README states liveness is future work: "proving safety (and, in the future, liveness) properties" ([S5]). | Veil evaluated on a "large set of case studies from the distributed system verification literature" ([S6]); "16 distributed protocol case studies. All 16 verified. Ivy failed on 2." ([S58], [S59]). |
| Veil README labels itself a "pre-release" with "a few bugs and rough edges" (fetched summary of [S5]). | Veil provides "push-button verification for decidable fragments of first-order logic" plus interactive Lean proofs ([S5], [S6]). |
| LeanMachines "not directly compatible" with Event-B tooling ([S1]). | Veil's VC generator has "a formal soundness proof" ([S6]). |

---

## 3. TLA-style encodings and Lean ports

### Facts recorded

- Leslie ([S13]): "Leslie is a shallow embedding of the Temporal Logic of Actions (TLA) in Lean 4." Capabilities listed: refinement mappings (Abadi-Lamport) with stuttering; multi-action specs (`ActionSpec`); CIVL-style layered refinement with mover types / Lipton reduction; round-based algorithms in the Heard-Of (HO) model; cutoff theorems for symmetric threshold protocols; random simulation. Requires Lean v4.27.0 (fetched). 416 commits (fetched).
- Leslie example status table (fetched): CounterRefinement, TwoPhaseCommit (10 actions), TicketLock, KVStore "Complete"; Paxos (14 actions) "Spec complete, 2 sorry"; VRViewChange "1 sorry"; LeaderBroadcast, FloodMin, BallotLeader, OneThirdRule, OneThirdRuleCutoff "Complete". Model-checking tactic and zero-one rule are "plans", not completed features.
- Leslie WF1 and weak-fairness rules exist (repo tree shows `WF.lean Weak fairness (WF1 rule)`) ([S13] excerpt); the fetched summary notes the README "does not report a specific liveness theorem for the listed protocol examples".
- lean-tla ([S14]): "Exploration workspace for a deep embedding of TLA+ in Lean 4, with Isabelle/TLA+ as the reference". Prototype includes behaviors, `□`/`◇`/`↝`, `[A]_v`, `WF`/`SF`, an invariant tactic, and "machine-checked examples (counter, refinement, ticket-lock liveness, two-process mutex with full-chain liveness)". Also proves Rule 6 and Theorem 1 of McMillan's relational rankings (CAV 2024) and a timestamped-queue example. Current focus: TLA meta-theory first.
- tla-plus-lean-workflow ([S39] excerpt): "Raft Leader Election: all three theorems fully proved. ElectionSafety holds vacuously (VotesGrantedBound makes LeadersHaveQuorum unreachable in 3-node model). Single-Decree Paxos: fully proved via five auxiliary invariants ... and majority_intersect lemma." Combines TLA+ (TLC) and Lean.
- compositional-temporal-safety ([S40] excerpt): "SAFETY is proved; LIVENESS is bounded-only. Lean proves only `□` safety. A liveness property ... is checked in TLC, on one small instance, under weak fairness - this is bounded evidence, NOT a proof, and NOT in Lean."
- Quint's tool reference lists, as one entry, "Models are TLA+-style transition systems embedded in Lean 4, so one model can be model checked, proved with SMT automation, and finished with interactive proofs where needed" ([S68], secondary; the entry's name was not captured in the excerpt, so attribution to a specific tool is unconfirmed).
- Lentil's embedding: "The definitions of the embedding are initially ported from `coq-tla`. The design of the proof mode is partly inspired by Iris Proof Mode." ([S15]).
- Federated verification paper: Raft implementation validated with TLAPS proofs (">eleven thousand lines") and cross-backend citation to "Coq, Lean 4, Why3, Apalache symbolic, CBMC ... and standalone Z3" ([S61]).

| Recorded items pointing toward "cannot" | Recorded items pointing toward "can" |
|---|---|
| Leslie Paxos: "Spec complete, 2 sorry"; VRViewChange "1 sorry" ([S13]). | Leslie: TwoPhaseCommit, TicketLock, KVStore, OneThirdRule marked "Complete" ([S13]). |
| lean-tla is an "Exploration workspace" ([S14]). | lean-tla has machine-checked ticket-lock and mutex liveness examples ([S14]). |
| Lean proves only safety in one composed-system repo; liveness checked in TLC instead ([S40]). | tla-plus-lean-workflow: single-decree Paxos "fully proved" in Lean ([S39]). |
| TLAPS-based Raft validation uses Lean 4 only as one of several cited backends ([S61]). | Same paper cites Lean 4 `#print axioms` closure checks as an acceptable discharge mechanism ([S61]). |

---

## 4. Temporal logic libraries; safety vs liveness

### Facts recorded

- LeanLTL: "a unifying framework for linear temporal logics in Lean 4 ... supports reasoning about traces that represent either infinite or finite linear time ... traditional LTL syntax to be combined with arbitrary Lean expressions" ([S17]). Embeds LTL and LTLf ([S18]).
- Lentil: "a proof infrastructure for linear temporal logic in Lean 4. It formalizes the semantics of LTL, provides rules and a dedicated proof mode to conduct proofs in LTL as an object logic." Operators include `□`, `◇`, `◯`, `𝑈`, `↝` (leads-to, "defined as `□ (p → ◇ q)`"), `Enabled`, `𝒲ℱ` (weak fairness) ([S15], [S16]).
- LeanearTemporalLogic: "syntax and semantics of LTL, transition systems, and ... various lemmas and theorems about LTL and LT properties" ([S19]).
- Other Lean LTL uses seen in results (search excerpts only): Sparkle hardware library "LTL operators (`always`, `eventually`, `next`, `Until`) ... for safety/liveness/fairness proofs directly against Signal code" (https://reservoir.lean-lang.org/@Verilean/sparkle, [S] 3rd-party package [X]); distributed_synthesis_lean defines LTL (https://github.com/BioDisCo/distributed_synthesis_lean [P/X]); LeanGuard uses LeanLTL for policy semantics (https://arxiv.org/pdf/2611.02332v1 [P/X]).
- Liveness status in distributed-protocol Lean work:
  - Veil: safety now; liveness "in the future" ([S5]).
  - ConsensusVerifier / AI-greement: "currently targets safety only" (fetched, [S38]).
  - lean-verification-components: Multi-Paxos "Liveness is not proved; progress requires additional leader, quorum, delivery and scheduler assumptions"; Raft joint-consensus: "Arbitrarily many reconfigurations, snapshots, Byzantine faults, crash/recovery and liveness remain outside scope." ([S35]).
  - raft-lean-squad: liveness and network partition tolerance listed as not covered (fetched, [S34]).
  - verified-consensus: "Liveness also needs participation and proposer recurrence"; assumptions include "bounded delivery after GST" ([S36]).
  - lean-dag: abstract states "machine-checked safety, liveness from eventual DAG synchrony" ([S37]).
  - lean-tla and Leslie include fairness/leads-to rules and some liveness examples ([S13], [S14]).
- Background on fairness (non-Lean): "a liveness property is only ever proved relative to a fairness assumption" ([S66], secondary).

| Recorded items pointing toward "cannot" | Recorded items pointing toward "can" |
|---|---|
| Several Lean protocol developments explicitly exclude liveness ([S5], [S34], [S35], [S38]). | Lean libraries exist with leads-to, weak/strong fairness, and WF1-style rules ([S13], [S14], [S16]). |
| Liveness proofs, where present, rely on stated environment assumptions (GST, eventual synchrony) ([S36], [S37]). | lean-dag states liveness proven; lean-tla proves ticket-lock/mutex liveness ([S37], [S14]). |
| Lentil/LeanLTL excerpts show libraries and logic embeddings; the excerpts do not show a protocol-implementation case study ([S16], [S17]). | LeanLTL paper says it provides "examples illustrating the utility of the library in reasoning about systems that come from applications" ([S17]). |

---

## 5. Process calculi and bisimulation

### Facts recorded

- CSLib: "We formalise Milner's CCS ... expressive enough to study concurrency patterns like semaphores" and prove "the hallmark result that bisimilarity is a congruence for CCS" ([S22]).
- CSLib HML: includes "syntax, satisfaction relation, and denotational semantics of HML, as well as a complete metatheory including the Hennessy-Milner theorem - bisimilarity coincides with theory equivalence for image-finite LTSs" ([S23]); applies directly to CCS and automata via the shared LTS API.
- CSLib scope statement: formalisations "include ... concurrency theory"; Pillar 2 roadmap includes "support for reasoning about concurrency" for the Boole framework, and Pillar 1 efforts in 2027 "will move on to technically challenging topics such as complexity theory, concurrency, secure compilation" ([S21]).
- CSLib status markers: v0.1.0 package on Reservoir (https://reservoir.lean-lang.org/packages, [P/X]); "the project is currently only a few months old" (roadmap section, [S21]); official launch announced 20 Feb 2026 ([S24]).
- A tracked listing mentions CSP, pi-calculus, Petri nets as concurrency models in CSLib (third-party summary https://www.emergentmind.com/topics/cslib [S/X]); the primary excerpts in [S22]/[S23] explicitly show CCS only. The pi-calculus is described in [S23] as motivation for a more complex HML variant, not as already formalised in CSLib.
- A Lean formalisation of CSP-style deadlock freedom is mentioned in the ITP 2026 volume listing ("All of our results have been formalized in the interactive theorem prover Lean on top of mathlib", with Isabelle/HOL also named in the abstract; https://drops.dagstuhl.de/entities/volume/LIPIcs-volume-382 [P/X]); the excerpt is ambiguous about which prover hosts which results.

| Recorded items pointing toward "cannot" | Recorded items pointing toward "can" |
|---|---|
| CSLib's concurrency work is stated as early; 2027 plans list concurrency as a "technically challenging" next topic ([S21]). | CCS, bisimulation, HML, congruence results are formalised on a general LTS API ([S22], [S23]). |
| One concrete CCS example in the excerpts is a vending machine ([S23]); no network-protocol CCS case study seen. | CSLib's LTS API is shared with automata and is intended for downstream reuse ([S23]). |

---

## 6. Session types, choreographies, protocol types

### Facts recorded

- Telltale: "The Lean project is an active mechanized proof stack. It covers session foundations, semantics, protocol coherence, runtime adequacy, and bridge theorems." ([S29]). Lean libraries `SessionTypes` and `SessionCoTypes` formalise MPST with `GlobalType`, `LocalTypeR`, bisimulation `EQ2`, and asynchronous subtyping (SISO decomposition) ([S30], secondary).
- Rumpsteak-Aura: "Formal verification of choreographic projection using Lean 4"; a `lean-exporter` "Exports choreography ASTs and projected programs to JSON for formal verification in Lean 4"; checks "Projection correctness ... Duality ... Subtyping invariants ... Effect conformance" ([S31]). Rust types "have a direct counterpart in the Lean formalization" (docs.rs rumpsteak_types, https://docs.rs/rumpsteak-types/latest/rumpsteak_types/ [P/X]). The mechanism is export of Rust ASTs to Lean, i.e. Lean checks exported data rather than Rust source.
- "Session Type State Spaces Form Lattices": "The paper's results have been mechanically verified in Lean 4 using Mathlib" with an "empirical study on 108 protocols" ([S32]).
- BSPL/CBCL (agent communication; secondary summary https://agent-comms.anuna.io/concepts/bspl/ [S/X]): "Lean 4 formalization (≈5,400 lines, 16 modules, 176 machine-checked theorems, zero sorry gaps ...)" and a "verified parser binary `cbcl-parse` is extracted from the proofs".
- Non-Lean comparator: mechanised MPST soundness/liveness proofs in Rocq: "we present the first mechanised proof of liveness for synchronous multiparty session types in the Rocq Proof Assistant" (ITP volume listing) and the ECOOP 2025 asynchronous MPST mechanisation ([S33]). The excerpts retrieved did not surface a comparable standalone Lean 4 MPST liveness-by-typing proof.

| Recorded items pointing toward "cannot" | Recorded items pointing toward "can" |
|---|---|
| Rumpsteak/Telltale bridge goes through JSON export of Rust ASTs; no direct Lean check of Rust source is described ([S31]). | Lean-based projection/duality/subtyping checks are in a working pipeline (`just rumpsteak-lean-check`) ([S31]). |
| Mature MPST mechanisations found in Rocq, not Lean ([S33]). | Telltale presents a Lean stack through "runtime adequacy, and bridge theorems" ([S29]). |

---

## 7. Distributed and consensus protocol developments in Lean 4

### Facts recorded

| Project | What the source states | Source |
|---|---|---|
| raft-lean-squad (Rust Raft, automated "Lean Squad" agents) | "716 theorems, 79 Lean files, 0 sorry, machine-checked by Lean 4.30.0-rc2 (stdlib only)"; headline safety theorem `fullProtocolStep_safe`; models abstract u64 as `Nat`, `HashMap` as function, omit "full message-passing network model", message loss, concurrency, I/O, liveness; "Lean models capture the algorithms faithfully (confirmed by 625 `#guard` assertions) but are not mechanically extracted from Rust source." | [S34] (fetched) |
| lean-verification-components | Raft joint consensus: reachable Election Safety, Log Matching, Leader Completeness for one old-to-joint-to-new transition; operational Multi-Paxos slot Agreement; "this is not a certification of a deployed Raft implementation." | [S35] |
| verified-consensus (Ethereum decoupled consensus) | "A Lean 4 model of the decoupled consensus protocol (Section 7 of the paper) and a machine-checked proof of its consensus guarantees"; premises include unforgeable signatures, bounded delivery after GST, honest committee majorities; some results "proved in ReviewTheorem.lean, but they are not fields of Consensus". Also an `AI_AUDIT.md` doc exists. | [S36] |
| lean-dag | Lean 4 + Mathlib formalisation of uncertified DAG consensus (Mysticeti-style); "machine-checked safety, liveness from eventual DAG synchrony". | [S37] |
| ConsensusVerifier (AI-greement) | ~135K lines of Lean covering Multi-Paxos, FastPaxos, SwiftPaxos, EPaxos*, Bodega and A-greement; proves linearizability of protocol models; "our Lean proofs cover a protocol model whose correspondence to the prototype is additionally checked by a human consensus expert"; "currently targets safety only". | [S38] (fetched) |
| tla-plus-lean-workflow | Raft leader election (3 theorems) and single-decree Paxos proved in Lean; Raft ElectionSafety "holds vacuously" in the 3-node model. | [S39] |
| Leslie | See section 3 (Paxos with 2 sorry; OneThirdRule etc. complete; cutoff theorems for all n). | [S13] |
| Veil | CAV'25: verifies a suite of benchmarks including Paxos, Vertical Paxos, Reliable Broadcast, Suzuki-Kasami, Stellar Consensus Protocol port; "Veil verifies all but 2 benchmarks in under 15 s (87.5%), and all but 4 benchmarks in under 10 s (75%)." Compared with Ivy: Ivy "succeeds in verifying Reliable Broadcast and Suzuki-Kasami and times out for Paxos and Vertical Paxos" on non-EPR encodings. | [S6] |
| Chronos Vault (vendor blog) | Claims "184 theorems" in Lean 4 for a 2-of-3 multi-chain protocol; the quoted theorem is `consensus_requires_two_validators (op) : hasConsensus op → op.confirmations.length ≥ 2`. Marketing context; the quoted theorem is a threshold property rather than a message-passing model. | [S69] (secondary) |
| BaDaaS survey (secondary) | States "A Lean port is in progress in cslib" for Raft, and that Verdi's Coq proof "runs to ~50000 lines". The CSLib sources read ([S20]-[S26]) do not themselves confirm a Raft port in CSLib. | [S67] |
| Lean Zulip (CSLib) | Verdi and ConCert named as "Real-World Verification Projects (as case studies/inspiration)" for porting from Rocq; "Rocq has ~40 years of CS formalization that Lean largely lacks". | [S25] (an AI-style summary post in the archive; treat as discussion, not a roadmap) |
| Raft paper (non-Lean) | "We have developed a formal specification and a proof of safety ... using the TLA+ specification language" (context only). | [S65] |

| Recorded items pointing toward "cannot" | Recorded items pointing toward "can" |
|---|---|
| Each consensus development above carries stated scope exclusions (liveness, crash/recovery, membership, network model, Byzantine) ([S34], [S35], [S36], [S38]). | Multiple independent Lean 4 developments report complete safety proofs for Raft/Paxos-family protocol models ([S34], [S35], [S38], [S39]). |
| Models are "not mechanically extracted" from the Rust code ([S34]); proofs "cover a protocol model" ([S38]); "not a certification of a deployed Raft implementation" ([S35]). | Veil reports automatic verification of Paxos/Vertical Paxos beyond what Ivy handled, and interactive proofs of SCP ([S6]). |
| Leslie Paxos has 2 `sorry` ([S13]). | Cutoff theorem reducing all-n verification to n ≤ K is "Fully proved" in Leslie (fetched, [S13]). |
| Two of the largest items (S34, S38) are described as agent-generated; S38 states proofs still need human review of model fidelity. | S34 reports 0 sorry; S38 reports kernel-checked proofs for several protocols. |

---

## 8. Refinement

### Facts recorded

- LeanMachines supports "relational refinement (Event-B)", "functional abstraction (lift)", and "strong refinement for algorithmic refinement" ([S1]). Paper: "functional refinement is most often applicable, and is thus undoubtedly the preferred refinement model in the LeanMachines framework." ([S2]). Example: refining a bounded `Bounded` counter machine to a bounded message "queue" with glue invariant ([S2]).
- Leslie: Abadi-Lamport refinement mappings with stuttering; Counter, TwoPhaseCommit "Refinement with invariant"; FloodMin "refinement to consensus spec"; CIVL-style layered refinement ([S13]).
- lean-tla: refinement example listed ([S14]).
- Cedar: the Lean model is the reference and the Rust implementation is related to it by testing, not by a Lean refinement proof ([S42], [S43]).
- Veil's lessons paper: Veil can "specify, implement, test and prove"; "bounded model checking to test that the protocol admits non-trivial executions" ([S8]).

| Recorded items pointing toward "cannot" | Recorded items pointing toward "can" |
|---|---|
| In the sources read, refinement proofs are between Lean-level models; the relation from a Lean model to code in another language is by testing (Cedar, raft-lean-squad, Zero-to-QED) rather than a refinement theorem ([S34], [S41], [S42]). | Refinement between machine models is available as a library discipline with proof obligations ([S1], [S2]) and as TLA refinement mappings ([S13]). |
| LeanMachines "alpha" ([S1]). | LeanMachines has a refereed journal paper and IFM paper ([S2], [S3]). |

---

## 9. Concurrency models and program-level reasoning

### Facts recorded

- Iris-Lean: "Lean 4 port of Iris, a higher-order concurrent separation logic framework" with MoSeL proof mode, `IProp`, `HeapLang`, invariants, later credits ([S27]). Paper: "provides complete coverage of all of the non-experimental features included in Iris" ([S28]). Repo source files `Language.lean`, `Lifting.lean`, `Adequacy.lean` exist (search excerpt, https://github.com/leanprover-community/iris-lean/blob/99bec4bf/Iris/Iris/ProgramLogic/Language.lean [P/X]), where `PrimStep` has threads forking `List Expr`. (Earlier repo state: "In its current stage, the scope of this project is to provide a framework to support proofs in different separation logics ... porting MoSeL", https://github.com/larsk21/iris-lean [P/X]; the later README lists HeapLang.)
- Loom / Velvet (program verifiers in Lean): monadic shallow embedding with "Non-Determinism semantics" ([S11], Reservoir: https://reservoir.lean-lang.org/@verse-lab/Loom [P/X]); Velvet supports "mutable state, loops, and non-determinism" ([S11] abstract via https://dl.acm.org/doi/10.1007/978-3-032-32526-6_11 [P/X]). Velvet v2.0: "rebuilt on Lean's new weakest-precondition verification infrastructure and `vcgen`" with no dependency on Loom or Mathlib ([S12]). The Loom paper states Veil 2.0 is built on Loom, "now featuring executable semantics" ([S11]).
- Aeneas (Rust to Lean): "no concurrent execution: long-term effort. We plan to address coarse-grained parallelism as a long-term goal." ([S48]); "A fundamental limitation is that the pure translation ... can also not support (safe) concurrent code, interior mutability or I/O" (Son Ho writeup, https://www.sonho.fr/assets/documents/aeneas.html [P/X]); Aeneas "currently targets a safe, sequential Rust code" ([S50]); other tool README: limitations "will be lifted by the ongoing work on separation logic: unsafe code, concurrency" (https://github.com/zksecurity/aeneas-pp/blob/main/README.md [P/X]).
- Lean runtime / library concurrency: reference manual section "Asynchronous Programming" exists ([S51]); `Std.Async` etc. A third-party project states "none of it is verified" and "no formal semantics of Lean's `IO`/`Task` exists at all" ([S53]). LeanIn offers a Lean 4 runtime "with machine-checked proofs" for a pure scheduler model, whose bridge theorems "name the primitive specifications they consume as axioms (`clock_spec`, `lock_spec`, ...)" ([S52]).
- Lean 2021 Zulip on network programming: "Currently there is no support for network programming, is that something that would eventually be supported by the standard library" with the answer "a separate Lean package sounds like the more likely option" ([S63]); later official plan slides mention Std scope "Async IO, Networking, Internet protocols" (Lean FRO community meeting PDF, https://leanprover.zulipchat.com/user_uploads/3121/0BIRP_WW_q6rvGqQdYRPKOSm/Monthly-Community-Meeting-10-May-2024.pdf [P/X]); [S54] shows an HTTP/1.1 state machine now in core.
- Zulip 2021 on systems programming: "being a reference-counted language, Lean needs the allocator in its runtime"; runs on Raspberry Pi; a micro-kernel in Lean "would be a work-intensive project" ([S62]).

| Recorded items pointing toward "cannot" | Recorded items pointing toward "can" |
|---|---|
| Aeneas does not support concurrency ([S48], [S50]). | Iris-Lean provides a concurrent separation logic with HeapLang ([S27], [S28]). |
| No formal semantics of Lean `IO`/`Task` is claimed to exist; Lean runtime concurrency unverified ([S53]). | Velvet/Loom handle effectful nondeterministic programs and Veil 2.0 has executable semantics ([S11], [S12]). |
| CSLib concurrency reasoning for code is on the roadmap, not delivered ([S21]). | CSLib has process-calculus and LTS infrastructure ([S22], [S23]). |
| LeanIn bridge theorems rest on axiomatised primitive specs ([S52]). | Lean core includes async primitives and a protocol state machine ([S51], [S54]). |

---

## 10. Linking models to implementations (code checking)

### Facts recorded

- Cedar (AWS): "verification-guided development": "writing an executable model of the system and mechanically proving properties about the model; writing production code for the system and using differential random testing (DRT) to check that the production code matches the model; and using property-based testing (PBT) to check properties of unmodeled parts" ([S42]). Lean model 1,673 LOC, proofs 5,714 LOC, Rust 24,915 LOC ([S43]). "uncovering nearly two dozen bugs" via DRT (OOPSLA paper excerpt, https://dl.acm.org/doi/10.1145/3649835 [P/X]). The domain is an authorization language, not a distributed protocol.
- Zero-to-QED chapter ([S41], fetched): Lean model of a circuit breaker; Rust `step` checked against it on "83,300 bounded single-step cases" and "2,025 trace tests"; "The tests provide evidence of correspondence, not a proof for all inputs"; limitations include "manual transcription", "Lean naturals are unbounded, whereas Rust integers are fixed-width", and "whether they capture the intended service behavior remains a specification question."
- raft-lean-squad ([S34]): `#guard` assertions between Lean model and Rust tests; "concrete function-level discharge" proposed as future work to "reduce the trust boundary."
- ConsensusVerifier ([S38]): "The authors distinguish formalizing from pseudocode from formalizing from implementation code, saying they present 'substantially different challenges'"; "Whether to formalize the implementation's behavior directly or develop implementation and specification from shared pseudocode remains 'an open question.'"
- Leslie repo tree includes `Rust/CoreSemantics.lean` ("Lean-side semantics for the Rust protocol core") and `Rust/RuntimeSemantics.lean`; the fetched summary notes the page "does not explicitly claim that a particular executable Rust implementation has been formally proved equivalent to a Lean model" ([S13]).
- Aeneas path (sequential Rust to pure Lean, then extrinsic proofs): "Once the Rust code is in Lean, we write a formal specification ... and prove that the extracted function matches it" (Runtime Verification blog, https://runtimeverification.com/blog/from-rust-source-code-to-mathematical-proof [S/X]); SymCrypt-scale cryptographic verification with Aeneas described in https://arxiv.org/html/2609.15648v1 [P/X] ("Aeneas has limited support for unsafe Rust and does not support raw pointers, interior mutability, or concurrency; functions using unsupported features must be modelled by hand").
- Signal Shot: "a public moonshot to verify the Signal protocol and its Rust implementation using Lean ... Aeneas, Mathlib and CSLib, ... `grind` ... AI" ([S58]); stated as an initiative in the FLoC 2026 slides; results not located in this pass.
- DyLean: protocol specs and security properties in the symbolic (Dolev-Yao) model, "allow for executable specifications"; case studies include "protocols involving unbounded loops or datastructures" and "protocols using merkle trees, ratcheting protocols, post-quantum protocols" ([S45], [S46], [S47]). The excerpts do not describe linking to an implementation; the talk calls it "still an early work-in-progress" (Dec 2025 slides, [S47]).
- CSLib Pillar 2: Boole IVL for "reasoning about code in mainstream programming languages ... (e.g., C/C++, Python, Rust)"; "verification conditions are generated as Lean goals" ([S26], [S20]); roadmap says that by end of 2027 CSLib "will have enabled the end-to-end verification of at least one substantially sized real-world system" ([S21]).
- Lean's own `HTTP/1.1` state machine exists as library code ([S54]); the retrieved excerpt does not state whether theorems about it exist.
- Agent-skill write-up (secondary): "A proof covers the model, not the code. Every finding ends in the real system" ([S70]).
- Veil's own lessons paper: Veil can specify, implement and test transition systems and run concrete/symbolic model checking from the same Lean file ([S8]); the README excerpts do not describe code extraction ([S5]).

| Recorded items pointing toward "cannot" | Recorded items pointing toward "can" |
|---|---|
| For every protocol-level Lean development read, the connection to real code is by testing, human review, or manual transcription, not by a Lean refinement/extraction proof ([S34], [S38], [S41], [S42]). | Lean models are executable and used as oracles against production code (Cedar DRT, Zero-to-QED, raft-lean-squad `#guard`) ([S41], [S42], [S34]). |
| Aeneas lacks concurrency; protocol implementations commonly involve concurrency or I/O ([S48], [S50]). | Aeneas gives proof-level (not test-level) correspondence for sequential safe Rust ([S48], [S49]); Veil/Velvet verify imperative/effectful programs in Lean ([S11], [S12]). |
| Specification-fidelity risk is named repeatedly ([S38], [S41]). | Cedar reports the approach found bugs and is deployed in a CNCF sandbox project (https://aws.amazon.com/blogs/opensource/cedar-joins-cncf-as-a-sandbox-project/ [P/X]). |
| Signal Shot results not yet seen; Boole concurrency/pointers are future work ([S21], [S58]). | CSLib/Boole design: VCs as Lean goals for code in mainstream languages ([S26]). |

---

## 11. Security/cryptographic protocol modeling

### Facts recorded

- DyLean: "a framework for the symbolic analysis of cryptographic protocols in the Lean theorem prover ... flexible general-purpose symbolic semantics, as well as a concrete proof methodology ... based on trace invariants" ([S46]). Artifact lists transition system, reachability definition, Hoare triples for send/receive/random, "key reachability theorem", "attacker knowledge theorem" ([bobdylean-paper-artifact](https://github.com/BobDyLean/bobdylean-paper-artifact) [P/X]). Methodology compared to loop invariants: "Design trace invariant / Preserved by each protocol step / Implies security properties" ([S47]).
- Tool landscape statement in the same abstract: Tamarin and ProVerif "provide automation"; DY* "hardcode threat models" ([S46]).
- LeanDY at PLAS 2026: "LeanDY: Type-Based and Trace-Based Symbolic Protocol Verification in Lean" (https://plas26.github.io/ [P/X]).
- Lean Squad/other claims of protocol proofs on blockchain topics (e.g. FOCIL, https://github.com/rahulbarmann/focil-lean4 [P/X]: "under Ethereum's standard '>2/3 honest validators' assumption, a transaction listed in a stored, non-equivocating inclusion list cannot be excluded from a canonical block") are existence records only.

| Recorded items pointing toward "cannot" | Recorded items pointing toward "can" |
|---|---|
| DyLean is described as early-stage in the 2025 talk ([S47]). | A refereed-venue symbolic protocol framework exists in Lean with case studies on looping protocols ([S46]). |
| Symbolic (Dolev-Yao) model is an abstraction of the protocol, not code ([S47]). | Two Lean symbolic-protocol frameworks at one workshop (DyLean, LeanDY) ([PLAS 2026](https://plas26.github.io/)). |

---

## 12. Lean 3 lineage

### Facts recorded

- Lean 3 documentation itself framed verification of protocols as an aim: "claims that pieces of hardware or software, network protocols, and mechanical and hybrid systems meet their specifications" ([S56]); same sentence in the Lean 2 system description (https://lean-lang.org/papers/system.pdf [P/X]).
- Lean 3 tactic framework paper (ICFP 2017) is the Lean 3 metaprogramming reference ([S55], [S57]).
- Community pages describe Lean 3 as obsolete: "This webpage is about Lean 3, which is effectively obsolete; the community has migrated to Lean 4." ([S55]).
- Several Lean 4 protocol-adjacent libraries state a non-Lean-3 lineage: Lentil ported from `coq-tla` ([S15]); Iris-Lean is a port of Rocq Iris ([S27]); CSLib discussion cites Rocq libraries (Verdi, ConCert) as inspiration ([S25]); Event-B (LeanMachines) ([S1]).
- Zulip 2021: Lean 3 vs Lean 4 for production software ("Lean 4 is several miles ahead of Lean 3 for practical production programming", Johan Commelin, https://leanprover-community.github.io/archive/stream/113489-new-members/topic/Writing.20Production.20Software.html [P/X]).
- Searches in this pass did not surface a Lean 3 distributed-protocol, TLA, or consensus verification project. Absence in search is not evidence of absence (see Open Questions).

| Recorded items pointing toward "cannot" | Recorded items pointing toward "can" |
|---|---|
| No Lean 3 protocol-verification project located; Lean 3 flagged obsolete ([S55]). | Lean 3 documentation lists protocols among intended verification targets ([S56]). |
| Lean 4 protocol libraries cite Coq/Rocq/Event-B, not Lean 3, as lineage ([S1], [S15], [S27]). | — |

---

## 13. Trust base and soundness notes recorded

- Lean 4.34.0 release notes: "three soundness issues, found with AI adversarial testing, have been analyzed and fixed, and a series of additional defensive checks have been implemented for further reinforcement." ([S60]).
- Developments commonly report `sorry` counts and axiom audits: raft-lean-squad "0 sorry"; Leslie "2 sorry"/"1 sorry"; LeanIn lists axioms for primitive specs; federated verification uses `#print axioms` ([S34], [S13], [S52], [S61]).
- Lentil's agent appendix and many repos have AI-agent authoring instructions (https://github.com/verse-lab/Lentil/blob/main/docs/Lentil-agent-appendix.md [P/X]); S34 and S38 describe agent-generated proofs checked by the Lean kernel.
- Veil: "Veil's VCG is proven sound w.r.t. the specification language semantics" ([S58]); Velvet's chain "is checked by the kernel" while Dafny "trusts its VC generator and an SMT solver" (Marktoberdorf lecture, https://leodemoura.github.io/static/marktoberdorf2026/lecture4/index.html [P/X]).

---

## 14. Quick tallies (descriptive only)

- Lean 4 libraries/frameworks directly for transition systems or protocols: LeanMachines, Veil, Leslie, lean-tla, Lentil, LeanLTL, CSLib LTS/CCS/HML, DyLean, Telltale (session types), Iris-Lean (program logic).
- Lean 4 protocol-specific case-study repos located: raft-lean-squad, lean-verification-components, verified-consensus, lean-dag, tla-plus-lean-workflow, compositional-temporal-safety, ConsensusVerifier, plus vendor blog S69.
- Cases in which the Lean artifact is formally tied to code by a Lean-checked proof (not just testing/review): Aeneas-extracted sequential Rust ([S48], [S49]); Velvet/Veil-embedded programs whose semantics are the Lean programs ([S11], [S12]); Rumpsteak exports checked in Lean ([S31]). Cases tied by testing/review: Cedar, raft-lean-squad, Zero-to-QED, ConsensusVerifier.

---

## 15. Open Questions

1. Does Veil (or Veil 2.0 on Loom) support extraction or refinement from a verified transition system to executable code in another language, and is there a published protocol implementation checked that way? The README excerpt reviewed did not describe code extraction ([S5]); the Loom paper says "executable semantics" ([S11]).
2. What exactly does "Veil 2.0 ... executable semantics" cover (concrete model checking of the Lean model only, or code that can run as the implementation)? Needs the full Loom paper Section 7.
3. Is there a Lean 4 proof of a liveness property for a full message-passing consensus protocol (not round-based/HO model), with explicit fairness and network assumptions? lean-dag and verified-consensus claim liveness under assumptions; neither was read in full ([S36], [S37]).
4. Is the Raft port inside CSLib real? One secondary source says it is "in progress" ([S67]); primary CSLib sources read do not confirm it.
5. Do any Lean 4 projects model network semantics (lossy, reordering, duplicating channels) as reusable libraries, as opposed to per-protocol encodings? Not located in this pass.
6. What is the Lean 4 status of Signal Shot (protocol plus Rust implementation)? Only an announcement-level mention was found ([S58]).
7. Does any project connect Aeneas-extracted Rust functions to a Lean protocol model (for example, proving that Rust `step` refines a Veil/Leslie `Next` relation)? Not located. raft-lean-squad names this as future work ([S34]).
8. Is there any Lean 3 distributed-protocol or TLA-style development? Searches here found none; Lean 3 community archives (Zulip #general, #lean) and `leanprover-community` repos were not exhaustively searched.
9. How do Lean 4 `Task`/`IO` semantics relate to formal models: is a formal semantics of Lean's runtime concurrency in progress ([S53] says none exists)?
10. What is the quantitative maturity of LeanMachines (usage, case studies beyond Event-B/VDM textbook examples)? Only two example sets seen ([S4]).
11. For Leslie, what is the status of the Rust semantics directories (`Rust/CoreSemantics.lean`, `Rust/RuntimeSemantics.lean`) and the stated "revised Rust verification architecture"? Only directory names were seen ([S13]).
12. How reproducible and independently audited are the large agent-generated developments (S34, S38)? The reports are self-published; no third-party audit was located.
13. Which of the primary excerpts cited as [X] would change if read in full (particularly S2, S3, S6, S8, S11, S17, S27, S28, S35-S37, S53)? A follow-up pass should fetch those in full.
14. Do the Lean 4 `Std.Http` protocol state-machine sources ([S54]) carry proofs (e.g., of parser or state invariants), and is there any formal specification linking them to RFC text?
15. Does a Lean-hosted, CSP-style deadlock-freedom or refinement-checking development exist (the ITP 2026 listing mentions one in an ambiguous Lean/Isabelle context)?
