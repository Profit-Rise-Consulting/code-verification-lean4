# REF-L04: Veil (verse-lab) and its protocol case studies

Access date for all sources: 2026-11-07. Source IDs refer to section 14.

## 1. Identity
| Field | Value |
|---|---|
| Subject (protocol or algorithm) | Framework for specifying, model checking and proving safety and liveness of state transition systems, aimed at distributed protocols. Ships case studies including Paxos (EPR and first-order versions), Vertical Paxos, NOPaxos, Bracha reliable broadcast (two encodings), ring leader election, two-phase commit, Stellar Consensus Protocol (SCP), Suzuki-Kasami, Ricart-Agrawala, Chord ring maintenance, multisig, blockchain, FloodSet, plus 29 TLA+-ported models that are model checked only. |
| Spec it targets (e.g. RFC 8446 section) | No RFC. Protocol models are ports of existing Ivy, mypyvy and TLA+ specifications (S1 section 4; file headers in S4), not of a standard or of production code. |
| Organisation / authors | VERSE Lab, National University of Singapore. CAV 2025 paper: George Pîrlea, Vladimir Gladshtein, Elad Kinsbruner, Qiyuan Zhao, Ilya Sergey (S1). |
| Primary URLs | https://github.com/verse-lab/veil ; https://veil.dev/ ; CAV 2025 paper https://doi.org/10.1007/978-3-031-98682-6_2 (PDF https://ilyasergey.net/assets/pdf/papers/veil-cav25.pdf); CAV artefact https://doi.org/10.5281/zenodo.15208271 |
| Repo, commit SHA, last commit date, licence | verse-lab/veil @ `57447722e7f25f06c75cff8735d10d41697afc14`, 2026-11-07 06:15:53 +0800, Apache-2.0. The public `main` history is a single squashed commit (`git log` shows 1 commit). |
| Lean toolchain (or other tool and version) | `leanprover/lean4:v4.35.0`. Dependencies pinned in `lake-manifest.json`: lean-smt (rev tag `v4.35.0-veil-no-mathlib`), Loom (verse-lab/loom @ `f0ea9100bec2e48afad7193053d6a89db3d57335`, a non-master snapshot dated 2026-11-06), Lentil (@ `1ddf1fc`), Lasso, VerifiedHAMT. SMT back ends: cvc5 (via lean-smt) and Z3. CAV 2025 evaluation used Lean 4.16, cvc5 1.2.1, Z3 4.14.0 (S1). |
| Maturity | Research prototype under active development ("Veil 2.0"; Veil 1.0 tagged `v1.0.0`). (A Lean FRO use-case page exists but was not read; see section 15.) AI provenance: `agent-config-present` (`.claude/CLAUDE.md`; `.agents/skills/tla-to-veil/SKILL.md`, a skill for translating TLA+ into Veil; README section "Agent skills"). This does not show who wrote the proofs, and the single squashed commit hides any trailers. Evidence level: `author-reported` for paper numbers, `static-inspected` for repository facts. |

## 2. Claim as stated by the authors (verbatim)
- README (S4): "Veil is a _foundational_ framework for (1) specifying, (2) implementing, (3) model-checking, and (4) proving safety and liveness properties of state transition systems, with a focus on distributed protocols." and "provides push-button verification for transition systems and their properties expressed in decidable fragments of first-order logic, with the full power of a modern higher-order proof assistant for when automation falls short."
- CAV 2025 (S1): "Veil is foundational: its verification condition (VC) generator is proven sound with respect to the semantics of its specification language."
- CAV 2025 evaluation (S1): "Veil verifies all but 2 benchmarks in under 15 seconds (87.5%), and all but 4 benchmarks in under 10 seconds (75%). Ivy times out for 2 of the benchmarks."
- Dafny 2026 workshop paper (S2) on trust: "users of Veil only have to trust the Veil frontend (i.e., syntax elaboration), the Lean compiler (which is used to execute the frontend code), and the Lean kernel" (this statement is for the proof-reconstruction mode; see section 6).

## 3. What is actually formalised
Modelling style. A Veil `module` declares abstract types (`type node`), relations and functions over them (the state), immutable background theory (`immutable individual`, `instantiate tot : TotalOrder round`), an `after_init` block, and `action`s written as imperative monadic code with `require`, nondeterministic `pick`/`:|`, and field updates. Message passing is modelled by relations such as `one_a R RP` or `echo_net S D V`; byzantine adversaries by a relation `is_byz` with quorum-intersection axioms as background theory. There is a second "distributed DSL" where one writes "the program each node runs" plus `network (synchrony := async)` assumptions; it "compiles such a description into an ordinary Veil module" (docs/Distributed-DSL.md, S4). Time and probability are not modelled.

Semantics and soundness (S4):
- Transition systems: `Veil/Core/Tools/ModelChecker/TransitionSystem.lean` defines `structure RelationalTransitionSystem (ρ σ l)` with fields `assumptions : ρ → Prop`, `init : ρ → σ → Prop`, `tr : ρ → σ → l → σ → Prop`.
- Invariance rule, `Veil/Core/Tools/Synthesizer/Soundness.lean`:
  ```lean
  def Invariant (p : ρ → σ → Prop) : Prop := ∀ th s, sys.reachable th s → p th s
  theorem invariant_of_inductive (hinit : ∀ th s, sys.assumptions th → sys.init th s → p th s)
      (hstep : ∀ th s lbl s', sys.assumptions th → p th s → sys.tr th s lbl s' → p th s') :
      sys.Invariant p
  ```
- Action semantics: `Veil/Frontend/DSL/Action/Semantics/Theorems.lean:246`:
  ```lean
  theorem VeilM.toTransitionDerived_sound (act : VeilM m ρ σ α) :
    act.toTransition = act.toTransitionDerived
  ```
  i.e. the transition relation derived via Loom's weakest-precondition calculus equals the operational one. The Loom POPL 2026 paper explains that Veil 2.0 checks "for each action both that (a) it does not throw any exceptions (assuming the invariant holds in the pre-state) and that (b) if it does not throw any exceptions, it preserves the invariant" and that the same action is interpreted under several semantics (demonic for invariants, angelic for symbolic bounded model checking) (S3 section 7.1).

Case-study statements (verbatim, `Examples/` at the commit above):
- Paxos, first-order (non-EPR) version, `Examples/Ivy/PaxosFirstOrder.lean:99-122`:
  ```
  safety [coherence] decision N1 R1 V1 ∧ decision N2 R2 V2 → V1 = V2
  invariant [unique_proposals] proposal R V1 ∧ proposal R V2 → V1 = V2
  invariant [vote_proposal] vote N R V → proposal R V
  invariant [decision_from_quorum] ...
  invariant [one_b_max_vote_properties1..3] ...
  invariant [none_vote] ¬ vote N none V
  invariant [choosable_proposal] ...
  ```
  closed by `#check_invariants` at line 130. A separate `Examples/Ivy/PaxosEPR.lean` encodes Paxos in the decidable EPR fragment.
- Vertical Paxos, `Examples/Ivy/VerticalPaxosFirstOrder.lean:132-174` (safety plus 19 auxiliary invariants), e.g.
  ```
  safety [coherence] decision N1 R1 V1 ∧ decision N2 R2 V2 → V1 = V2
  invariant [unique_config] configure_round_msg R C1 ∧ configure_round_msg R C2 → C1 = C2
  invariant [propose_if_lower_configured] proposal R2 V ∧ tot.le R1 R2 → ∃ c, configure_round_msg R1 c
  invariant [join_ack_msg_property3] join_ack_msg N R RP V ∧ V ≠ none → vote N RP V
  ```
  `#check_invariants` at line 348.
- Reliable broadcast, distributed-DSL version, `Examples/Distributed/ReliableBroadcast.lean:78-95`:
  ```
  safety [agreement]
    ¬ is_byz A ∧ ¬ is_byz B ∧ delivered A V1 ∧ delivered B V2 → V1 = V2
  safety [integrity]
    ¬ is_byz originator ∧ ¬ is_byz N ∧ delivered N V → V = input
  invariant [echo_once] ¬ is_byz N → echoed N V1 ∧ echoed N V2 → V1 = V2
  invariant [vote_once] ¬ is_byz N → voted N V1 ∧ voted N V2 → V1 = V2
  ```
  Global-action version: `Examples/Ivy/ReliableBroadcast.lean:122-197` (safeties `vote_integrity`, `deliver_integrity`, `agreement`; 10 invariants).
- Ring leader election, `Examples/Ivy/Ring.lean:42-45`:
  ```
  safety [single_leader] leader N ∧ leader M → N = M
  invariant [leader_greatest] leader L → le N L
  invariant [self_msg_greatest] pending L L → le N L
  invariant [drop_smaller] pending S D ∧ btw S N D → le N S
  ```
  with `set_option veil.smt.trust false` at line 47 (proof reconstruction on) and `#model_check { node := Fin 4 } { }` at line 55.
- NOPaxos (fixed leader, no view change or synchronisation), `Examples/Ivy/NOPaxos.lean:243`: `safety [consistency] gh_committed S V1 ∧ gh_committed S V2 → V1 = V2`.
- Stellar Consensus Protocol, `Examples/StellarConsensus/SCP.lean:218-220`:
  ```
  safety [intertwined_safe]
    ∀ (n1 n2 : node) (b1 b2 : ballot) (v1 v2 : value),
      intertwined n1 ∧ intertwined n2 ∧ confirmed_committed n1 b1 v1 ∧ confirmed_committed n2 b2 v2 → v1 = v2
  ```
  Background properties are a type class `SCP.Background` (line 18) whose docstring says "In the Ivy spec, they appear as `trusted` properties (assumptions)"; S1 says these were proved in Lean from a concrete model (ported Isabelle proofs) so the class is instantiated.
- Two-phase commit, `Examples/Ivy/TwoPhaseCommit.lean:85-95`: `safety (decide_commit N → ¬decide_abort N2) ∧ (decide_commit N -> vote_yes N2) ...` with 8 invariants named `manual_1`..`manual_8`.
- Liveness, `Examples/Liveness/TLA/Consensus.lean:47`: `temporal (only := verify) [success] 𝒲ℱ choose → ◇ ⌜ someChosen ⌝`, proved with `prove_temporal_by [success]` using Lentil's `tapply wf1_original` (line 69) with `set_option veil.smt.trust false` (line 39) and an in-file axiom audit via `Lean.collectAxioms ``success._proof` (line 80). `Examples/Demo/FloodSet/FloodSetLiveness.lean` declares `temporal [terminates]`, `[decisionStable]`, `[everyoneDecides]` (lines 78, 82, 86), model checks them with `#temporal_check`, and proves only `decisionStable` (line 102).
- TLA+ ports (`Examples/TLA/*.lean`, 29 files: Raft, MultiPaxos, EPaxos, Paxos, Percolator, CosmosDB, AWSDnsRace, etc.): each uses `#model_check` (1-6 per file); only `Peterson.lean` also runs `#check_invariants`. These are finite-instance explicit-state model checks, not proofs.

The CAV 2025 Rabia and SuzukiKasamiInts case studies are not present in the current `Examples/` tree (searched with `rg -il rabia`).

## 3a. Predicate semantics
Headline results are the `#check_invariants` outcomes for each case study (S4 @ 57447722e7f25f06c75cff8735d10d41697afc14). Conclusion predicates, verbatim:
- Reachability and invariance, `Veil/Core/Tools/ModelChecker/TransitionSystem.lean:49-51` and `Veil/Core/Tools/Synthesizer/Soundness.lean:32,38`:
  ```lean
  inductive reachable (sys : RelationalTransitionSystem ρ σ l) (th : ρ) : σ → Prop where
    | init : ∀ (s : σ), sys.assumptions th → sys.init th s → sys.reachable th s
    | step : ∀ (s s' : σ), sys.reachable th s → sys.next th s s' → sys.reachable th s'
  def Invariant (p : ρ → σ → Prop) : Prop := ∀ th s, sys.reachable th s → p th s
  theorem invariant_of_inductive (hinit : ∀ th s, sys.assumptions th → sys.init th s → p th s)
      (hstep : ∀ th s lbl s', sys.assumptions th → p th s → sys.tr th s lbl s' → p th s') :
      sys.Invariant p
  ```
- Per-action VC predicates, `Veil/Frontend/DSL/Action/Semantics/Definitions.lean:206-243`:
  ```lean
  def VeilM.succeedsAndMeetsSpecification (act : VeilM m ρ σ α) (pre : SProp ρ σ) (post : RProp α ρ σ) : Prop :=
    [DemonFail| triple pre act post]
  def VeilM.doesNotThrowAssuming (act : VeilM m ρ σ α) (assu : ρ → Prop) (pre : SProp ρ σ) : Prop :=
    VeilM.succeedsAndMeetsSpecification act (fun th st => assu th ∧ pre th st) ⊤
  def VeilM.succeedsAndPreservesInvariantsAssuming (act : VeilM m ρ σ α) (assu : ρ → Prop) (inv : SProp ρ σ) : Prop :=
    VeilM.succeedsAndMeetsSpecification act (fun th st => assu th ∧ inv th st) (fun _ => inv)
  ```
- What `#check_invariants` emits (docstring, `Veil/Frontend/DSL/Module/Syntax.lean:412-415`): "Theorems use the VC names in the current namespace, e.g. `keep_excluded` for action `keep` and invariant `[excluded]`, or `keep_doesNotThrow` for its assertion check. VCs without a successful proof witness are skipped".
- Strongest premises: the module's `assumption`s, which are background axioms on immutable state. One example is quorum intersection, `assumption ∀ (Q1 : quorum) (Q2 : quorum), ∃ (N : node), member N Q1 ∧ member N Q2` (`Examples/Ivy/PaxosFirstOrder.lean:27`). Abstract sorts may have any cardinality. Unless `veil.smt.trust false` is set, the SMT solver's `unsat` answers are also premises (section 6). `trusted invariant` is a language feature that assumes facts about mutable state (`Syntax.lean:290-299`); `rg` found no use of it in `Examples/`.

Plain language. Guaranteed, modulo the trust items in section 6: in every state reachable from an initial state through the modelled actions, under the stated assumptions, the safety property holds. This holds for every instantiation of the abstract sorts. Not guaranteed: anything about deployed code; liveness, unless proved separately; and properties of models that are only model checked, because `#model_check` covers one finite instance such as `{ node := Fin 4 }`. Static inspection found no single top-level theorem of the form `sys.Invariant Invariants` emitted per module. The recorded result is the set of per-VC theorems. Assembling them into `Invariant` would follow the `invariant_of_inductive` rule; this was not confirmed by a build.

## 4. Property classes covered
| Class | Status | Evidence |
|---|---|---|
| Functional correctness | no (protocol-level only) | Specs are abstract transition systems. |
| Safety invariant | proved (SMT trusted by default) | Inductive invariants for 16 CAV case studies (S1); current `#check_invariants` files listed in section 3. |
| Liveness | proved for 2 small examples only | WF1-based proofs for small examples (Consensus, one FloodSet property) via Lentil (S4, S5). Fairness is "stated directly as a premise" (S5). No liveness proof for Paxos/Raft-class protocols found in the repo. |
| Refinement | no | No refinement theorem between models or to implementations found. S5 footnote: "Refinement is an important topic in TLA+, but it is orthogonal to what we focus on in this post." |
| Symbolic security | partial | Byzantine-fault models (reliable broadcast, SCP) with `is_byz` relations and quorum assumptions; no Dolev-Yao cryptographic attacker. |
| Computational security | no | |
| Parsing | no | |
| Constant-time | no | |
| Other | tested (model checking, simulation) | Explicit-state and symbolic bounded model checking, invariant synthesis (`#updr`), temporal model checking (`#temporal_check`), randomised simulation verified sound in Loom (S3 section 7). |

## 5. Link to implementation code
Category: none to external code; the model itself is executable in Lean.
- Actions have "runtime execution semantics" (S3 section 7) and are compiled for the explicit-state model checker; the distributed DSL produces per-node programs, but these remain Lean definitions inside a Veil module. No extraction to, or conformance checking against, Rust/Go/C++ implementations was found in the repo or papers.
- The case studies are ports of existing Ivy/TLA+ models; the guarantee is about the model, not about any deployed system.

## 6. Trusted computing base and soundness hygiene
Keyword counts across all 89,474 Lean lines (`rg -w`, includes comments, docstrings and test files): `sorry` 37, `admit` 10, `axiom` 21, `native_decide` 22, `implemented_by` 16, `extern` 0, `opaque` 8, `partial` 84.
Refined: zero `axiom` declarations (`rg "^\s*(private |protected |noncomputable )?axiom "` returns 0); the `axiom` hits are uses of `collectAxioms` and prose. Non-comment `sorry|admit` hits: 10 in `Veil/`, 1 in `Examples/`, 9 in `VeilTest/`. The `Veil/` hits are (a) a generated suggestion template `unveil; sorry` for failed VCs (`Veil/Core/Tools/Verifier/Manager.lean:191`), (b) the synthesiser's `admit` function name, and (c) `r.admit` in `Veil/Core/Tools/Synthesizer/Solver.lean:277`, documented as "Steps lean-smt cannot reconstruct remain as goals; they are admitted, so the proof carries `sorry` and the answer counts as trusted." `native_decide` occurs as the default model-checker discharge tactic `first | decide | native_decide` (`Veil/Frontend/DSL/Module/Elaborators/Core.lean:672`).

Key trust switch: `Veil/Base.lean:144-148`:
```lean
register_option veil.smt.trust : Bool := {
  defValue := true
  descr := "If true, `veil_smt` trusts unsat results from the SMT solver. \
  If false, `veil_smt` asks the SMT backend to reconstruct Lean proofs."
}
```
So by default SMT `unsat` answers are accepted without a kernel-checked proof; Veil prints "Trusting SMT solver for {count} {goalWord}. `set_option veil.smt.trust false` to enable proof reconstruction." (`Veil/Frontend/DSL/Module/Elaborators/Verification.lean:226`). S1 footnote 3: "Currently, proof reconstruction (provided by Lean-SMT) is off by default in Veil." S2: "Veil supports proof reconstruction (at a 3-5x performance penalty) using Lean-SMT". Only some example files set `veil.smt.trust false` (e.g. `Ivy/Ring.lean:47`, `Liveness/TLA/Consensus.lean:39`).
Other TCB items: Veil frontend elaboration (unverified meta-code, per S2), Lean compiler, Lean kernel, cvc5/Z3 when trusted; model-checker results rely on compiled code and `native_decide`. Background theory (e.g. quorum intersection, total orders) enters as assumptions; for SCP these are discharged by an instance.

## 7. Scale, effort, automation
- CAV 2025 (S1): 16 case studies (9 from IvyBench, 2 Paxos variants from Padon et al., 5 others), "1704 non-empty non-comment Lean lines, 85 actions, 185 invariants" in total; hardware MacBook (M4, 32 GB); 5 s per-SMT-query timeout (120 s for Rabia); Ivy given 300 s; average of 5 runs. Per-benchmark timing figure (Fig. 5) is a chart whose numbers did not extract from the PDF text; the textual summary is quoted in section 2. Non-EPR benchmarks: Paxos, Vertical Paxos, SuzukiKasamiInts, ReliableBroadcast; Ivy (complete=fo) times out on Paxos and Vertical Paxos.
- Loom POPL 2026 (S3): "The Veil 2.0 benchmark suite contains 17 specifications of 15 different distributed protocols".
- Dafny 2026 (S2): about 18 months of development; "Veil is more than 10x slower than Ivy ... our largest case study taking 160 seconds, compared to Ivy's 50 seconds."
- Repo size at commit: 89,474 Lean lines total (`Veil/` 46,443; `Examples/` 18,120; `VeilTest/` 23,949; `Benchmarks/` 602). Individual case studies are 63-350 lines (e.g. VerticalPaxosFirstOrder 350, NOPaxos 271, SCP 316 plus SCPTheory 132, PaxosFirstOrder 132, Ring 69).
- Automation: SMT (cvc5, Z3) on first-order VCs with finite-model-finding; interactive Lean fallback; invariant synthesis; Lentil tactics for temporal goals.
- Not built here (section 10), so no local timings.

## 8. Limitations stated by the authors (verbatim)
- S2: "Veil is more than 10x slower than Ivy"; "If you plan to build an auto-active verifier in Lean, expect to spend most of your time on performance engineering."
- S2 on shallow embedding: it "might make it unsuitable for implementing verifiers for languages that are too different from Lean due to either being untyped (e.g., TLA+) or because their memory model and a type system are too different from the one of Lean (e.g., C)."
- S1: "Currently, proof reconstruction (provided by Lean-SMT) is off by default in Veil."
- S5: "for now we model fairness by stating it directly as a premise of the temporal property"; "Many mechanised protocol developments prove safety in full and then leave liveness as future work".
- S3 (NOPaxos): "Our simplified model of the protocol omits these components [view change and synchronisation], so we consider only the case with a fixed leader."

## 9. Independent evidence
- Bugs/discrepancies found by the authors: S1 reports for Rabia "we spotted one discrepancy between an invariant in the Ivy formulation and the corresponding admitted one in the Rocq code". S3 reports that a verified-sound randomised simulator showed NOPaxos action `handle_gap_commit_reply` "appeared to never be taken" because "one of its require statements was always false"; the authors "fixed this action along with the affected invariant clauses and several related actions, and ultimately certified the safety property of the corrected specification". These are bugs in models, not in deployed code.
- Adoption: listed as a Lean FRO use case (https://lean-lang.org/use-cases/veil/, found via search, not read in full; see section 15). No independent third-party review was read.

## 10. Reproducibility
Not built: sandbox disk limit; queued for CI reproducibility job. A first attempt (`cd /tmp/veil && timeout 1500 lake build`, chained after Lentil/Velvet) did not reach Veil: the shared sandbox ran out of space while elan extracted `leanprover/lean4:v4.35.0` ("No space left on device (os error 28)"), and builds were then stopped on instruction. Exact command for the CI job:
```
git clone --depth 1 https://github.com/verse-lab/veil && cd veil && git checkout 57447722e7f25f06c75cff8735d10d41697afc14 \
  && curl https://raw.githubusercontent.com/leanprover/elan/master/elan-init.sh -sSf | sh -s -- -y --default-toolchain none \
  && timeout 1500 lake build
```
The CAV 2025 artefact is archived at https://doi.org/10.5281/zenodo.15208271 (not downloaded).

## 11. Generalisation
Pattern: an Ivy-style, SMT-backed, decidable-fragment protocol verifier shallowly embedded in Lean, with a VC generator proved sound in Lean and an interactive fallback. To apply it to an arbitrary codebase one would need (a) a hand-written abstract model of the protocol in Veil's relational DSL, (b) inductive invariants (manual or synthesised), and (c) a separate refinement or conformance mechanism to the code, which Veil does not provide. Preconditions specific to this case: protocols whose state fits first-order relations over abstract sorts (ideally EPR); assumptions such as quorum intersection can be axiomatised in a type class; performance is acceptable for models of tens to hundreds of lines.

## 12. Bearing on sub-claims (facts only, no verdict)
| Sub-claim | Fact from this reference | Source ID |
|---|---|---|
| H1 | Paxos, Vertical Paxos, NOPaxos, Bracha broadcast with byzantine nodes, SCP and 29 TLA+ models (Raft, EPaxos, Percolator...) are expressed as Lean transition systems; message passing via relations and a node-program DSL; no time or probability. | S1, S3, S4 |
| H2 | Inductive-invariant safety proofs for 16 (CAV) / 17 (POPL) specs, 1704 lines in total, most in under 15 s; by default SMT results are trusted, reconstruction costs 3-5x; liveness only for small examples. | S1, S2, S4, S5 |
| H3 | No link from Veil models to implementation code in a mainstream language was found; models are executable in Lean only. | S3, S4 |
| H4 | Authors say the shallow-embedding approach may be unsuitable for untyped languages or C-like memory models. | S2 |
| H5 | Veil's executable semantics and model checker allow testing and simulation of the model independently of proofs; no evidence of checking external implementations. | S3, S4 |
| D | "Verify" = SMT-discharged or kernel-checked inductive invariant; "model check" = finite-instance explicit-state or symbolic BMC; "foundational" = VC generator proved sound in Lean. | S1, S3 |

## 13. Conflicts
- Benchmark counts: S1 reports 16 case studies; S3 reports "17 specifications of 15 different distributed protocols" for Veil 2.0; the current repo lacks the CAV Rabia and SuzukiKasamiInts files and adds 29 model-checked TLA+ ports.
- Performance relative to Ivy: S1 reports Ivy timing out on 2 benchmarks that Veil verifies; S2 reports "Veil is more than 10x slower than Ivy ... 160 seconds, compared to Ivy's 50 seconds". These refer to different versions and possibly different benchmarks.
- Trust: S2 states users "only have to trust the Veil frontend ..., the Lean compiler ..., and the Lean kernel", while the shipped default `veil.smt.trust := true` (S4) also trusts the SMT solver unless the option is changed.

## 14. Sources
| ID | Title | URL | P/S | Depth | Summary |
|---|---|---|---|---|---|
| S1 | Veil: A Framework for Automated and Interactive Verification of Transition Systems (CAV 2025) | https://ilyasergey.net/assets/pdf/papers/veil-cav25.pdf | P | [FULL] | Design of Veil 1.0, soundness of VC generation, 16 case studies, comparison with Ivy. |
| S2 | Lessons from Building an Auto-Active Verifier in Lean (Dafny 2026 workshop) | https://verse-lab.org/papers/veil-dafny26.pdf | P | [FULL] | Experience report: 18 months, performance engineering, TCB, 3-5x reconstruction cost, >10x slower than Ivy. |
| S3 | Foundational Multi-Modal Program Verifiers (Loom, POPL 2026) | https://dl.acm.org/doi/10.1145/3776719 (PDF https://ilyasergey.net/assets/pdf/papers/loom-popl26.pdf) | P | [SECT] sections read: 1 Introduction, 2 Overview, 7 Veil 2.0 (incl. the NOPaxos case study), 8 Velvet, 9 Related work, 10 Conclusion; claims used here come from section 7 | Loom framework; section 7 rebuilds Veil as Veil 2.0, with a semantics zoo and the NOPaxos case study and bug. |
| S4 | verse-lab/veil repository @ 5744772 | https://github.com/verse-lab/veil | P | [CODE] @ 57447722e7f25f06c75cff8735d10d41697afc14 | Source of all file:line quotes, counts and options. |
| S5 | Liveness Proofs in Veil, Part I: The First Step (Zhao, Sergey, 2026-06-24) | https://proofsandintuitions.net/2026/06/24/liveness-proofs-in-veil-part-1/ | P | [FULL] | Walkthrough of WF1-based liveness proof of a one-step consensus model in Veil using Lentil. |

## 15. Unverified leads (excluded from evidence)
- Lean FRO use-case page "Veil: Multi-Modal Verification of Distributed Protocols", https://lean-lang.org/use-cases/veil/ (search result only).
- veil.dev publications page and online playground https://try.veil.dev/ (not read in full).
- Springer chapter page https://link.springer.com/chapter/10.1007/978-3-031-98682-6_2 (same paper as S1, read via the author PDF).
- CAV 2025 artefact https://doi.org/10.5281/zenodo.15208271 (not downloaded).
