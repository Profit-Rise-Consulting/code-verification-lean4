# REF-L19: Leslie (TLA in Lean 4, rupakm)

## 1. Identity
| Field | Value |
|---|---|
| Subject (protocol or algorithm) | A shallow embedding of TLA in Lean 4. Library features: refinement mappings, `ActionSpec` multi-action specs, CIVL-style layers and movers, Heard-Of round-based model, cutoff theorems for symmetric threshold protocols, random simulation, and a probabilistic layer (`Leslie/Prob`). The examples include Paxos (several variants), OneThirdRule, Ben-Or, BallotLeader, LastVoting, VR view change, 2PC, KV store, lease lock, fencing tokens, Chandy-Lamport, Byzantine reliable broadcast, binding crusader agreement and cache coherence |
| Spec it targets | Lamport TLA; Charron-Bost & Schiper Heard-Of model; textbook protocol descriptions. No RFC |
| Organisation / authors | Rupak Majumdar (commit author; MPI-SWS affiliation not stated in the repo). README: "Leslie is ported from the 'Lentil' implementation, which came out of the `coq-tla` library" |
| Primary URLs | https://github.com/rupakm/leslie ; Reservoir page https://reservoir.lean-lang.org/@rupakm/leslie (cited in lean-tla's research index, not fetched) |
| Repo, commit SHA, last commit date, licence | `rupakm/leslie` @ `58e58b2502b2a570e4ce4fdb6b40748c3cb30e5d` (Merge PR #116 "feat/fencing-tokens-demo"), 2026-06-09 01:17 +0200, Apache-2.0 |
| Lean toolchain (or other tool and version) | `leanprover/lean4:v4.27.0`; dependencies Mathlib, `lintLlmProofs` (https://github.com/jessealama/lint-llm-proofs), aesop, plausible |
| Maturity | Research library. AI provenance: `agent-config-present`. Determined from: `CLAUDE.md` ("# AI Agent Instructions"), a `.claude/` directory with `agents/`, `commands/` and `worktrees/`, `docs/superpowers/plans`, the `lintLlmProofs` lake dependency, and phase checkpoint files (`PHASE-8-5d-CHECKPOINT.md`) |

## 2. Claim as stated by the authors (verbatim)
README.md:
> "Leslie is a shallow embedding of the Temporal Logic of Actions (TLA) in Lean 4. It provides a framework for specifying and verifying concurrent and distributed systems with machine-checked proofs."

> "**Agreement for OneThirdRule** (`OneThirdRule.lean`): Fully machine-checked proof that the OneThirdRule consensus algorithm satisfies agreement for any number of processes n, under the 2/3-communication predicate."

> "**Cutoff theorem** (`Cutoff.lean`): For symmetric threshold protocols with communication closure, safety at all n reduces to checking n ≤ K where K = ⌈k·α_den/(α_den−α_num)⌉+1. Fully proved including the scaling lemma, partition sum, and weighted partition sum."

`docs/rust-verification.md`:
> "Given a round-based protocol proved safe in Leslie and a Rust implementation using the `Communication`/`InboxExt` API, establish that the Rust code correctly implements the Leslie model — and therefore inherits the Leslie safety proof."

## 3. What is actually formalised
OneThirdRule agreement over the round-based spec, for any `n` (`Leslie/Examples/OneThirdRule.lean:378`):
```lean
theorem otr_agreement :
    pred_implies (otr_spec n).toSpec.safety
      [tlafml| □ ⌜agreement n⌝] := by
  apply pred_implies_trans (q := [tlafml| □ ⌜lock_inv n⌝])
  · apply round_invariant
    · exact lock_inv_init n
    · exact lock_inv_step n
```
Single-decree Paxos refinement to an abstract consensus spec, for any `n` acceptors and `m` proposers with injective ballots (`Leslie/Examples/Paxos.lean:926`):
```lean
theorem paxos_refines_consensus {n m : Nat} (ballot : Fin m → Nat)
    (h_inj : Function.Injective ballot) :
    refines_via paxos_ref (paxos n m ballot).toSpec.safety consensus.toSpec.safety_stutter := by
  apply refinement_mapping_stutter_with_invariant
    (paxos n m ballot).toSpec consensus.toSpec paxos_ref (paxos_inv ballot)
```
Cutoff (`Leslie/Cutoff.lean:446`). This is a meta-theorem reducing a configuration invariant at all `n` to all `n ≤ cutoffBound`:
```lean
theorem cutoff_reliable
    (_alg : SymThreshAlg k α_num α_den) (hα : α_num < α_den)
    (h_half : 2 * α_num ≥ α_den)
    (inv : ConfigInv k)
    (h_thresh : inv.threshDetermined α_num α_den)
    (h_all_small : ∀ n, n ≤ cutoffBound k α_num α_den →
      ∀ c : Config k n, inv n c) :
    ∀ n (c : Config k n), inv n c
```
Environment model: TLA behaviours (`Nat → σ`) with stuttering. Communication in round-based examples is an HO collection constrained by a communication predicate, which covers lossy delivery. Interleaving examples use `ActionSpec` with gated actions. There is no Raft example. The KV-store example treats the replicated log as a black box: "In a real implementation the journal would be a consensus-based replicated log (e.g., Raft or Multi-Paxos); here we treat it as a black box" (`Leslie/Examples/KVStore.lean:15-17`).

## 3a. Predicate semantics
OneThirdRule. Conclusion, `Leslie/Examples/OneThirdRule.lean:174-178` @ `58e58b2`:
```lean
def agreement (s : RoundState (Fin n) LState) : Prop :=
  ∀ p q v w,
    (s.locals p).decided = some v →
    (s.locals q).decided = some w →
    v = w
```
Headline (`OneThirdRule.lean:378-380`): `pred_implies (otr_spec n).toSpec.safety [tlafml| □ ⌜agreement n⌝]`. The strongest premise is built into the spec's communication predicate (`OneThirdRule.lean:158-160`):
```lean
def comm_two_thirds : CommPred (Fin n) :=
  fun _r ho => ∀ p : Fin n,
    ((List.finRange n).filter fun q => ho p q).length * 3 > 2 * n
```
Plain language: in a Heard-Of round model where every process hears from more than 2n/3 processes in every round, decided processes never disagree. Not guaranteed:
- Behaviour when the communication predicate fails (for example message loss beyond that bound).
- Byzantine processes; the model is benign HO.
- Any link to the Rust code.

Paxos refinement. `paxos_refines_consensus` (`Leslie/Examples/Paxos.lean:926-928`) has premise `(h_inj : Function.Injective ballot)` and conclusion `refines_via paxos_ref (paxos n m ballot).toSpec.safety consensus.toSpec.safety_stutter`. `refines_via` (`Leslie/Refinement.lean:59-61`):
```lean
def refines_via {σ : Type u} {τ : Type v}
    (f : σ → τ) (concrete : pred σ) (abstract : pred τ) : Prop :=
  ∀ e, concrete e → abstract (exec.map f e)
```
The abstract `consensus` spec (`Paxos.lean:30-40`) has `init := fun s => s.chosen = none` and two actions, `choose1`/`choose2`. Each is gated on `s.chosen = none` and sets `chosen` to `some .v1` / `some .v2`.

Plain language: every execution of the Paxos model, mapped by `paxos_ref`, is a stuttering execution of a two-value "choose once" spec. So at most one value is ever chosen, under injective ballot assignment. Not guaranteed:
- More than two values in this example.
- Liveness.
- Implementation conformance.

Generic simulation. `SimulationRel.soundness` (`Leslie/Refinement.lean:382-386`) states that every concrete execution has a related abstract `safety_stutter` execution under a monotone time map. Its body is `sorry`. Results that go through it, rather than through `refinement_mapping_*`, are not closed (lexical observation; not a proof-closure audit).

Cutoff. `cutoff_reliable` (`Leslie/Cutoff.lean:446-452`) has premises `(hα : α_num < α_den) (h_half : 2 * α_num ≥ α_den)`, `(h_thresh : inv.threshDetermined α_num α_den)` and `(h_all_small : ∀ n, n ≤ cutoffBound k α_num α_den → ∀ c : Config k n, inv n c)`. Its conclusion is `∀ n (c : Config k n), inv n c`. Plain language: a threshold-determined predicate that holds on all configurations up to the cutoff holds for every n. The premise quantifies over all configurations, not only reachable ones.

Secrecy (added in Phase 1c for F-013). `shamir_secrecy` (`Leslie/Examples/Prob/Shamir.lean:165-173`):
```lean
theorem shamir_secrecy {t : ℕ}
    (partyPoint : Fin n → F)
    (h_nz_pp : ∀ i, partyPoint i ≠ 0)
    (h_F : t + 1 ≤ Fintype.card F)
    (C : Coalition n t) (s s' : F) :
    (Leslie.Prob.Polynomial.uniformWithFixedZero t s).map
        (fun f => fun (i : C.val) => some (f.eval (partyPoint i.val)))
      =
    (Leslie.Prob.Polynomial.uniformWithFixedZero t s').map
        (fun f => fun (i : C.val) => some (f.eval (partyPoint i.val)))
```
Plain language: for a uniformly random degree-t polynomial with constant term s, the joint distribution of the shares seen by a coalition of at most t parties (evaluation points ≠ 0) is the same for any two secrets. This is information-theoretic secrecy of one dealing step. It is not a protocol-execution secrecy theorem against an adaptive adversary, and not a computational result. The framework predicate `Secrecy` (`Leslie/Prob/Secrecy.lean:77`) quantifies over adversaries over traces. I did not check which examples instantiate it. The `Secrecy.lean` header says a framework sorry remains in `RandomisedAdversary.lean` (`randomisedTraceDist_eq_bind_traceDist`). However, the theorem at `RandomisedAdversary.lean:1296` has no lexical `sorry`, and the `Leslie/Prob` lexical count is 1 `sorry`, in `Spike/ASTSanity.lean` only. The header appears stale; this is a lexical observation, not a closure audit.

## 4. Property classes covered
| Class | Status | Evidence |
|---|---|---|
| Functional correctness | no | none |
| Safety invariant | yes | `otr_agreement`, Paxos `agreement`, BallotLeader, Ben-Or, VR view change, KVStore |
| Liveness | yes | `*Liveness.lean` examples (BallotLeader, AllGather, BindingCrusaderAgreement, Counter, KVStore, Peterson), `Rules/WF.lean` WF1 |
| Refinement | yes (with a gap) | `paxos_refines_consensus`, 2PC, FloodMin, CounterRefinement. The general `SimulationRel.soundness` is stated and sorry'd (Section 6) |
| Symbolic security | no | No symbolic (Dolev-Yao) model found in the inspected files (lexical search of `Leslie/Prob` and `Leslie/Examples/Prob` file names and theorem names) |
| Computational security | partial (information-theoretic secrecy only) | Proved theorems: `shamir_secrecy` (`Leslie/Examples/Prob/Shamir.lean:165-173` @ `58e58b2`) and `avss_secrecy` (`Leslie/Examples/Prob/AVSSAbstract.lean:2020-2031`, grid form; its docstring says full row+column view secrecy is "explicitly deferred"). No computational-hardness reduction was found; the subset proved is perfect (distributional-equality) secrecy of share views for coalitions of size ≤ t. See §3a |
| Parsing, constant-time | no | none |
| Other: parameterized cutoff | yes | `Cutoff.lean` |

## 5. Link to implementation code
Mechanism: none machine-checked. There is a Rust crate `rust/` (`src/comm.rs`, `protocol.rs`, `driver.rs`, `ballot_leader_phased.rs`; tests `one_third_rule.rs`, `lease_lock.rs`; 1,892 lines). The test header says "Rust implementation and tests of the OneThirdRule consensus protocol, corresponding to Leslie's `OneThirdRule.lean`." The planned architecture in `docs/rust-verification.md` has a `RustCorrespondence.lean` and Verus annotations. Its assumptions section says "We defer the question of how to discharge them (Verus solver, manual audit, or testing)" and "We do not verify the transport." No `RustCorrespondence.lean` exists at `58e58b2`. The README lists `Leslie/Rust/CoreSemantics.lean` and `RuntimeSemantics.lean`, but the directory `Leslie/Rust` does not exist. The current guarantee is testing only.

## 6. Trusted computing base and soundness hygiene
Static count over `Leslie/` and the root, with comments and strings stripped (165 `.lean` files under `Leslie/`, 103,720 lines; 1,521 `theorem`/`lemma`): `sorry` 5, `admit` 0, `axiom` 0, `native_decide` 1, `implemented_by` 0, `extern` 0, `opaque` 0, `partial def` 14.
- `Leslie/Refinement.lean:387`: `theorem SimulationRel.soundness` is "**Full soundness** (statement only)", proved by `sorry`. It is not referenced elsewhere by name.
- `Leslie/Refinement.lean:515`: backward-simulation invariant transfer, with the comment "For now, we sorry this and note that the backward simulation itself is the main contribution".
- `Leslie/Examples/LastVoting.lean:304` `lv_inv_preserved` and `:399` `proposals_respect_votes_invariant` are both `sorry`. `lv_inv_preserved` is used at line 317, so LastVoting's safety chain is incomplete.
- `Leslie/Prob/Spike/ASTSanity.lean`: 1 `sorry` (spike/sanity file).
- `native_decide`: `Leslie/Examples/Paxos/BoundedSingleProposer.lean:453 theorem verifyN3_ok : verifyN3 = true`.
TCB: the Lean kernel and Mathlib, plus the compiler for the one `native_decide`. The faithfulness of each Lean protocol model to the paper description is unverified.

## 7. Scale, effort, automation
- 165 Lean files and 103,720 lines under `Leslie/`; 1,521 theorems; 9,356 lines in `Leslie/Prob`.
- Automation: Claude Code agents (`.claude/agents`, `.claude/commands`), the `lintLlmProofs` linter dependency, and SageOx agent instructions in `CLAUDE.md`. Tactics include `action_cases` and `#gen_frame_lemmas` (CLAUDE.md "Leslie Gadgets"). Person-effort is not stated.
- Build time: not measured.

## 8. Limitations stated by the authors (verbatim)
- `Leslie/Examples/LastVoting.lean:392-394`: "The inductive structure means this proof needs a strengthened invariant that tracks the highest accepted ballot for each value across the entire execution history. We leave this as sorry."
- `Leslie/Refinement.lean:513-514`: "For now, we sorry this and note that the backward simulation itself is the main contribution; invariant transfer requires additional machinery."
- `docs/rust-verification.md`: "**Transport correctness assumed.** The `Transport` trait implementation satisfies: messages are not fabricated, duplicated, or corrupted. Messages may be lost or reordered. ... We do not verify the transport."

## 9. Independent evidence
None found. lean-tla (REF-L18) vendors Leslie as a reference artifact under `docs/research/vendor/leslie`, and its research index cites the Leslie Reservoir page. That shows awareness, not evaluation.

## 10. Reproducibility
Not built (orchestrator policy: shared sandbox disk was full, no toolchain or Mathlib downloads).

## 11. Generalisation
Pattern: a reusable TLA-in-Lean library with protocol-agnostic proof rules (refinement mapping with stuttering, round invariants, WF1, cutoff), so each protocol proof is a model plus an inductive invariant plus calls to library rules. The cutoff theorem is the generalisable piece. For symmetric threshold protocols it turns "all n" into a finite check, which the authors suggest could use model checking or `native_decide`. Requirements for an arbitrary codebase: someone writes a Lean protocol model by hand. The implementation link is planned (Verus annotations and a correspondence file) but not built. Specific to this case: Heard-Of round structure and threshold symmetry.

## 12. Bearing on sub-claims (facts only, no verdict)
| Sub-claim | Fact from this reference | Source ID |
|---|---|---|
| H1 | A general TLA-in-Lean library with refinement, liveness (WF1) and HO-model rules. Many consensus and broadcast protocols are modelled, but not Raft | S1 |
| H2 | 1,521 theorems with 5 sorry. The generic simulation soundness theorem and LastVoting invariant preservation are sorry'd. Paxos, OTR and cutoff have none | S1 |
| H3 | No Lean-to-Rust link exists in the repo. The planned Verus-based design and the README-listed `Leslie/Rust` files are absent | S1, S2, S3 |
| H4 | Developed with Claude Code agents and an LLM-proof linter (`lintLlmProofs`) | S1, S4 |
| H5 | Rust tests named as corresponding to Lean models, with no formal tie | S3 |
| D | README uses "Fully machine-checked" and "Fully proved" for OTR and cutoff | S2 |

## 13. Conflicts
- README status table: "Paxos | ... | Spec complete, 2 sorry" and "VRViewChange | ... | 1 sorry". Code at `58e58b2`: 0 `sorry` in `Paxos.lean` and `VRViewChange.lean`. README appears stale.
- README Project Structure lists `Leslie/Rust/CoreSemantics.lean` and `RuntimeSemantics.lean`. The directory does not exist.
- README Project Structure lists 11 examples. `Leslie/Examples` has more than 40 files (e.g. BenOr, LastVoting, ByzantineReliableBroadcast, FencingTokens).

## 14. Sources
| ID | Title | URL | P/S | Depth | Summary |
|---|---|---|---|---|---|
| S1 | rupakm/leslie @ 58e58b2: Lean sources (`Leslie/*.lean`, `Leslie/Examples/**`) | https://github.com/rupakm/leslie/tree/58e58b2/Leslie | P | [CODE] | TLA shallow embedding in Lean 4 with refinement, HO-round and cutoff rules plus 40+ protocol examples. 5 lexical sorry, including generic `SimulationRel.soundness` |
| S2 | `README.md` @ 58e58b2 | https://github.com/rupakm/leslie/blob/58e58b2/README.md | P | [CODE] | Project claims and status table (stale with respect to the tree; see Conflicts) |
| S3 | `docs/rust-verification.md` and `rust/` @ 58e58b2 | https://github.com/rupakm/leslie/blob/58e58b2/docs/rust-verification.md | P | [CODE] | Rust correspondence approach. The tree contains Rust tests only; `Leslie/Rust` and `RustCorrespondence.lean` are absent |
| S4 | `CLAUDE.md` and `.claude/` @ 58e58b2 | https://github.com/rupakm/leslie/blob/58e58b2/CLAUDE.md | P | [CODE] | Agent instructions ("# AI Agent Instructions"). Basis for the `agent-config-present` category |

Access date: 2026-11-07.

## 15. Unverified leads (excluded from evidence)
- Reservoir page https://reservoir.lean-lang.org/@rupakm/leslie was not fetched.
- `Leslie/Prob` beyond `Secrecy.lean`, `Shamir.lean:150-185` and `AVSSAbstract.lean:2005-2035` (coupling, randomised adversary and AVSS protocol-level theorems) was not inspected.
- Any paper by Majumdar describing Leslie was not searched for.
