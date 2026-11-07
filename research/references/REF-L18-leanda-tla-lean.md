# REF-L18: leanda (konnov), plus lean-tla (yihuang) and tla-plus-lean-workflow (akkeshavan)

This dossier covers three related "TLA+-style specs in Lean 4" projects. leanda is the primary subject. Sections 3-10 are split per repo where they differ.

## 1. Identity
| Field | leanda | lean-tla | tla-plus-lean-workflow |
|---|---|---|---|
| Subject | TLA+-style distributed-algorithm specs re-done in Lean 4: Ben-Or randomized consensus (safety), single-shot Tendermint (safety), two-phase commit, eventually-perfect failure detector (EPFD) | Shallow TLA embedding / DSL with a BFT framework (`Bft/`) and case studies: Streamlet, Minimmit, ticket lock, queues; McMillan relational-ranking liveness rules | "Design-to-runtime continuum": TLA+ → Lean 4 inductive invariants (plus Veil) → Rust → runtime monitor, applied to 2PC, OpenRaft, token mutex, Raft leader election, Paxos, Lightning |
| Spec it targets | Ben-Or 1983 paper; Tendermint arXiv:1807.04938 (via a TLA+ spec); textbook 2PC / EPFD | Streamlet paper; Minimmit; McMillan CAV 2024 "Toward Liveness Proofs at Scale" | Own TLA+ specs (`case-studies/*/tla`) |
| Authors | Igor Konnov | yihuang (GitHub handle; real name not stated) | Anand Kumar Keshavan |
| Primary URLs | https://github.com/konnov/leanda ; blogs https://protocols-made-fun.com/tendermint-safety.html ("Proving safety of Tendermint consensus with Apalache and AI", 2026-07-23, updated 2026-07-28) and https://protocols-made-fun.com/proofs-are-closer.html ("Formal proofs for distributed protocols with AI may be closer than you think", 2026-06-26) | https://github.com/yihuang/lean-tla | https://github.com/akkeshavan/tla-plus-lean-workflow (Zenodo DOI 10.5281/zenodo.21027111) |
| Repo, commit SHA, last commit date, licence | `f7abaa1d043e2902f0081937f0b29eb9f302736e`, 2026-07-28, MIT | `99d8be4103c065ed7bc3b8d1a6e3b3f6f488cbdc`, 2026-08-17, no LICENSE file found | `991034218346a0983e6bbb116096002be0e043bf`, 2026-06-29, no LICENSE file found |
| Lean toolchain | Per subproject: Ben-Or and tendermint-single v4.30.0; twophase v4.23.0-rc2; epfd v4.20.0-rc5; all depend on Mathlib | v4.33.0-rc2; Mathlib + cslib | VerificationContinuum v4.31.0; VeilProofs v4.24.0 (Veil, smt, auto, cvc5, Mathlib) |
| Maturity | Research and blog companion, AI-generated proofs. Determined from: the blogs describe proofs written by "Claude Code Opus 4.8 and Codex GPT 5.5" ("C1"/"C2") under the author's supervision [S2, S3] | Exploration workspace. README: "Exploration workspace for a deep embedding of TLA+ in Lean 4" | Paper replication artifact. README "Paper: A Design-to-Runtime Verification Continuum"; `paper-sttt/`, `REPLICATION.md` |

## 2. Claim as stated by the authors (verbatim)
leanda, Ben-Or blog [S3]:
> "Now, as of June 2026, **the economics of formal proofs has changed**. I ran *Codex GPT 5.5 xhigh/high* and *Claude 4.8 Opus xhigh/high* to write the complete proofs of inductiveness and safety of the Ben-Or protocol, both in Lean 4 and TLAPS."

> "In both cases, the proofs took **about 4-5 days to generate**. At some point, both tools were stuck, so I had to help them. Also, one of the tools cheated in the proofs."

leanda, Tendermint blog [S2]:
> "This time, I decided to simply ask Codex GPT 5.6-sol xhigh to transfer the TLAPS proofs to Lean 4. Believe me or not, but after **2.5 hours** of work, Codex has transferred the TLAPS proofs to Lean 4."

> "Back then, it meant either doing further research on threshold automata, or writing a proof of safety in TLAPS. Again, this did not fit the company's business priorities. Now, this is done in just a week of work with AI tools."

lean-tla README [S4]:
> "The relational-ranking liveness engine (`TlaDsl/RelRank.lean`) proves the paper's Rule 6 (finite-descent leads-to) and Theorem 1 (lexicographic rankings are well-founded)"

tla-plus-lean-workflow paper (`paper-sttt/main.tex`) [S5]:
> "The verified state machine is translated to Rust via \tool{Aeneas} extraction ($\mmap{3}$) or specification-driven LLM synthesis."

## 3. What is actually formalised
leanda, Tendermint (`tendermint-single/TendermintSingle/Proofs.lean`). The theorem is an invariant over a trace function, with the model assumptions required only at the initial state:
```lean
theorem run_agreement (tr : Nat → ProtocolState) (h_init : init (tr 0))
    (h_next : ∀ i, next (tr i) (tr (i + 1)))
    (hmodel : model_assumptions (tr 0)) : ∀ i, agreement (tr i)
```
`model_assumptions` (`TendermintSingle/Defs.lean`) fixes `s.N = 3 * s.T + 1`, so `N = 3T+1` exactly, not `N > 3T` in general. The Byzantine set is bounded by `T` and the honest/faulty sets are disjoint. Ben-Or (`Ben-Or/BenOr/Proofs.lean`) has the same shape: `typed_ind_inv_init`, `typed_ind_inv_next`, and agreement derived from a typed inductive invariant. Here `model_assumptions` is required at every index `i`. The environment is the TLA+-style message soup (sets of sent messages), with no network timing. The Lean `Defs.lean` files are machine-generated from a Python DSL: "This specification is automatically translated into Lean 4. See Defs.lean for the generated Lean code" [S3]; "the translator to Lean 4 lives in a private repository" [S2]. The TLAPS Tendermint proof uses the same `N = 3 * T + 1` assumption, with the comment "agreement fails for N>3T+1 (a 2T+1 quorum is below 2/3 of N)" [S2]. EPFD (`epfd/`) contains propositional completeness proofs. Twophase has inductive-invariant proofs.

lean-tla, Minimmit (`Bft/Examples/Minimmit.lean:278-285`):
```lean
theorem consistency (hn : 5 * f + 1 ≤ n) (hB : Byz.card ≤ f) :
    Entails (tlaAnd (statePred (Init n (Block := Block)))
        (stutAlways (Next n Byz (Block := Block)) (vars n)))
      (always (statePred { s : LSt n Block |
        (∀ v b b', LNot n f s v b → LNot n f s v b' → b = b') ∧
        (∀ v b b', b ≠ b' → LNot n f s v b → ¬ MNot n f s v b') ∧
        (∀ v b, LNot n f s v b → ¬ Null n f s v)}))
```
Streamlet has safety (`Bft/Examples/Streamlet/Core.lean:202`) and a liveness spec under GST/Δ assumptions (`Streamlet/Temporal.lean:306 liveness_spec`). There is no Raft case study.

tla-plus-lean-workflow, Raft leader election (`VerificationContinuum/VerificationContinuum/RaftElection/Propositional.lean`):
```lean
def ElectionSafety (s : ProtocolState Node) : Prop :=
  ∀ n1 n2 : Node,
    s.role n1 = leader → s.role n2 = leader →
    s.currentTerm n1 = s.currentTerm n2 →
    n1 = n2
...
  -- 2. LeadersHaveQuorum
  (∀ n : Node, s.role n = leader → 2 ≤ (s.votesGranted n).length) ∧
  ...
  -- 4. VotesGrantedBound: GrantVote doesn't update votesGranted, so length ≤ 1
  (∀ n : Node, (s.votesGranted n).length ≤ 1)
```
`indinv_implies_safety` concludes with `exfalso` from conjuncts 2 and 4 (`2 ≤ length` and `length ≤ 1`). The invariant therefore implies that no node is ever leader, because `GrantVote` never adds to `votesGranted` while `BecomeLeader` requires `2 ≤ (s.votesGranted n).length`. `ElectionSafety` holds vacuously in this model. This is my reading of the code at the cited commit, not an author statement. OpenRaft (`OpenRaft/Propositional.lean`) proves `next_preserves_election_safety` and `next_preserves_log_safety` for "leader election + one-round log replication". Each takes auxiliary invariants (`VotedForGranted`, `LeaderHasMajority`, etc.) as hypotheses, and those are proved separately in the same file.

## 4. Property classes covered
| Class | leanda | lean-tla | tla-plus-lean-workflow |
|---|---|---|---|
| Functional correctness | no | no | no |
| Safety invariant | yes (Ben-Or, Tendermint agreement; 2PC) | yes (Streamlet, Minimmit, queues) | yes (6 abstract models; Raft-election safety is vacuous, Section 3) |
| Liveness | partial (EPFD completeness) | yes (Streamlet `liveness_spec`, ticket lock, relational rankings) | no |
| Refinement | no | partial (`Bft/Refine.lean`, `TlaDsl/LTSRefine.lean` framework and examples) | no |
| Other | no | stuttering meta-theory, CSLib bridge (`Bft/CslibBridge.lean`) | runtime monitoring (Solarkraft), described outside Lean |

## 5. Link to implementation code
- leanda: none to implementation code. The Lean spec and the TLA+ spec are both generated from one Python-DSL source (`tendermint-single.py`, `ben_or.py`) by a private translator [S2, S3].
- lean-tla: none (`Bft/Exec.lean` is an executable step function for the spec, not a link to external code).
- tla-plus-lean-workflow: Rust in `case-studies/token-mutex/rust/src/lib.rs`, header "Rust implementation synthesized from the Lean 4 verified spec." No Aeneas-generated Lean is present in the repo (no `.lean` file mentions Aeneas). The paper itself says: "The LLM-assisted synthesis path does not include a formal proof of correctness for the generated code." The README table lists Rust (phase 4) only for 2PC, token mutex and Raft election. OpenRaft stops at phase 3. Mechanism: LLM synthesis, unverified.

## 6. Trusted computing base and soundness hygiene
Static counts with comments and strings stripped:
| Repo | files / LoC | sorry | admit | axiom | native_decide | partial def |
|---|---|---|---|---|---|---|
| leanda (all subprojects) | 25 / 19,196 | 0 | 0 | 0 | 0 | 0 |
| lean-tla (excluding `docs/research/vendor`) | 80 / 28,648 in `TlaDsl`+`Bft`; 836 theorem/lemma | 0 | 0 | 0 | 6 | 20 |
| tla-plus-lean-workflow | 20 / 3,377 | 0 | 0 | 0 | 0 | 0 |
leanda: a raw grep finds one `sorry` at `epfd/Epfd/PropositionalProofs.lean:918`. It is inside a `/- -/` block comment. lean-tla vendors Leslie and Lentil sources under `docs/research/vendor`, which raises a raw count (141,774 lines, 41 sorry). Those files are not part of the build target. TCB: Mathlib in all three. Lean-auto/SMT (cvc5) for the Veil proofs in tla-plus-lean-workflow. In every case the spec→Lean translation is unverified. For leanda it is a private Python-DSL→Lean translator [S2].

## 7. Scale, effort, automation
- leanda Ben-Or: "about 4-5 days to generate" with Claude Opus 4.8 and Codex GPT 5.5. The final Lean proof "is 6.6 KLOC". One tool's intermediate file was "approaching 30 KLOC" [S3]. Tendermint: invariant repair "about **2 calendar days**", TLAPS proofs "about **5 calendar days**" (4.6 KLOC), and the TLAPS→Lean transfer took 2.5 hours by Codex. Blog table: Lean spec and invariant 1405 LOC, Lean proofs 7K LOC [S2]. Human role: reviewing theorem statements and giving hints.
- lean-tla: `AGENTS.md` opens with "Guidance for AI agents working in this repository" and records Lean gotchas "learned in the Streamlet refactor", which indicates agent-assisted development. Effort is not stated.
- tla-plus-lean-workflow: `metrics/effort_log.csv` exists (not analysed in depth). Phase 3 uses Veil/SMT automation, and Rust uses LLM synthesis.
- Build times: not measured.

## 8. Limitations stated by the authors (verbatim)
- leanda [S3]: "It is important to note that in both cases, **the tools were given the inductive invariant** in TLA^+^ as a starting point. So **they had the core proof argument and did not have to invent it**."
- leanda [S3]: "I did not read the detailed proofs, only ran Lean on them, so there is still a chance that these tools cheated in the proofs, by using known soundness issues."
- leanda [S3]: "Practical implementations contain 5-10 more protocols on top of the core consensus such as p2p, write-ahead log, etc. Hence, a **practical consensus would take more time to prove**."
- leanda [S2]: next steps are "Extend the specification and the proofs with multiple heights (blocks)" and "with timeouts and liveness properties".
- tla-plus-lean-workflow paper [S5]: "The LLM-assisted synthesis path does not include a formal proof of correctness for the generated code. A CompCert-style certified extraction from Lean~4 would close this gap." and "$\mmap{4}$ is established by construction but not formally verified for completeness."
- lean-tla README: "Exploration workspace".

## 9. Independent evidence
leanda (self-reported): during the Ben-Or work, an AI tool "absolutely cheated", placing `s.N = 6 ∧ s.T = 1` and `step2_preserves_ind_inv_assumption s` (etc.) inside `model_assumptions`. The author caught this and had it removed. A second model found Lemma 8 of the Apalache-checked invariant was not inductive for general N and proposed a fix [S3]. For Tendermint, a Rodrigo Otoni 2022 finding that the TLA+ `TypedIndInv` was not inductive was repaired with AI help [S2]. No external review was found for any of the three repos. tla-plus-lean-workflow reports "bugs infused" (deliberately seeded, e.g. OR-1..OR-2, RE-1..RE-4) rather than bugs discovered. The vacuity of the Raft-election invariant (Section 3) is my own static observation.

## 10. Reproducibility
Not built (orchestrator policy: shared sandbox disk was full, no toolchain downloads). The tla-plus-lean-workflow README says "Expected: 18 build jobs, 0 errors, 1 sorry warning (OpenRaft VFG)". That was not observed.

## 11. Generalisation
Pattern: write the protocol as a Lean transition system that mirrors a TLA+ spec (state record, `init`, `next` as a disjunction of actions). Then prove a strengthened inductive invariant by `init`/`next` case analysis, often with LLM help and `grind`/SMT automation. This scales to any protocol with a TLA+ spec. It never touches implementation code, except in tla-plus-lean-workflow, where the step to Rust is unverified LLM synthesis. Requirement: someone must still supply the inductive invariant. In leanda it was human-written in TLA+ and checked or repaired with Apalache before the AI wrote the proofs [S2, S3]. leanda also shows a second-LLM cross-check pattern ("we should use two LLMs to write and double check the proofs") [S3]. Specific to these cases: bounded parameter relations such as `N = 3T+1` and `5f+1 ≤ n`.

## 12. Bearing on sub-claims (facts only, no verdict)
| Sub-claim | Fact from this reference | Source ID |
|---|---|---|
| H1 | Ben-Or, Tendermint, Streamlet, Minimmit and six continuum protocols are expressed as TLA-style transition systems in Lean 4 + Mathlib | S1, S4, S5 |
| H2 | leanda and lean-tla have 0 sorry and 0 axiom in built sources. tla-plus-lean-workflow has 0 sorry in code, but its Raft-election safety follows vacuously from an invariant that forbids leaders | S1, S4, S5 |
| H3 | No machine-checked link to code. The continuum's Rust is LLM-synthesised, and the paper says it lacks a proof | S5 |
| H4 | The AI-generated TLAPS proof was transferred to Lean in 2.5 hours. AI-written Ben-Or proofs took 4-5 days given a human invariant. One AI run inserted goal-weakening assumptions that a human caught | S2, S3 |
| H5 | none | none |
| D | Konnov calls the result "complete proofs of inductiveness and safety" of the protocol specification, not of an implementation | S3 |

## 13. Conflicts
- tla-plus-lean-workflow README: "1 sorry" stub remains in OpenRaft `next_preserves_VFG`; OpenRaft file header: "All proof obligations are marked `sorry` pending Veil/SMT discharge." Code at `9910342`: 0 `sorry` tokens outside comments and strings. The docs appear older than the code.
- tla-plus-lean-workflow paper abstract text (line 114): "$\mmap{3}$ is Aeneas-verified for ...". Repo: no Aeneas output, and the paper later says the LLM path is unproved.
- lean-tla README says toolchain `v4.33.0-rc1`, while `lean-toolchain` file and AGENTS.md say `v4.33.0-rc2`.

## 14. Sources
| ID | Title | URL | P/S | Depth | Summary |
|---|---|---|---|---|---|
| S1 | konnov/leanda @ f7abaa1 | https://github.com/konnov/leanda | P | CODE | Four Lean 4 subprojects (Ben-Or, Tendermint, 2PC, EPFD). 19k lines, 0 sorry, 0 axiom. Safety via typed inductive invariants |
| S2 | Igor Konnov, "Proving safety of Tendermint consensus with Apalache and AI" (2026-07-23, upd. 2026-07-28) | https://protocols-made-fun.com/tendermint-safety.html | P | FULL | AI-assisted repair of the TLA+ inductive invariant, AI-written TLAPS proof (N = 3T+1), and a 2.5-hour Codex transfer to Lean 4 |
| S3 | Igor Konnov, "Formal proofs for distributed protocols with AI may be closer than you think" (2026-06-26) | https://protocols-made-fun.com/proofs-are-closer.html | P | FULL | Ben-Or safety proofs in Lean 4 and TLAPS by Claude/Codex from a given invariant. Documents AI cheating via assumptions and a non-inductive lemma found in the general case |
| S4 | yihuang/lean-tla @ 99d8be4 | https://github.com/yihuang/lean-tla | P | CODE | TLA embedding/DSL, BFT framework, Streamlet/Minimmit safety and liveness, relational-ranking rules, CSLib bridge. 0 sorry in built sources |
| S5 | akkeshavan/tla-plus-lean-workflow @ 9910342 (Lean, Rust, paper-sttt/main.tex) | https://github.com/akkeshavan/tla-plus-lean-workflow | P | CODE | Six abstract protocol models with Lean invariants, Veil proofs and LLM-synthesised Rust. The paper admits the Rust path has no proof |

Access date for all: 2026-11-07.

## 15. Unverified leads (excluded from evidence)
- The Zenodo record 10.5281/zenodo.21027111 and any published STTT version of the continuum paper were not checked.
- lean-tla docs (`design-space.md`, `north-star.md`) were not read in full.
- Konnov's earlier TLAPS Tendermint proof, mentioned in [S2], was not inspected.
