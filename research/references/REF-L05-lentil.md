# REF-L05: Lentil (linear temporal logic / TLA proof infrastructure in Lean 4)

Access date for all sources: 2026-11-07. Source IDs refer to section 14.

## 1. Identity
| Field | Value |
|---|---|
| Subject (protocol or algorithm) | Library, not a protocol: a shallow embedding of LTL/TLA over infinite traces, proof rules (WF1, SF1, leads-to calculus), and a dedicated temporal proof mode. Used by Veil (REF-L04) for liveness. |
| Spec it targets (e.g. RFC 8446 section) | None. Targets Lamport's TLA rules (WF1/SF1). Definitions "initially ported from coq-tla" (S1 README). |
| Organisation / authors | VERSE Lab, NUS (verse-lab GitHub organisation). The Veil liveness blog post is by Qiyuan Zhao and Ilya Sergey (S3). No paper on Lentil itself was found. |
| Primary URLs | https://github.com/verse-lab/Lentil ; docs `docs/Lentil-reference.md`, `docs/Lentil-cheatsheet.md`, `docs/Lentil-agent-appendix.md` |
| Repo, commit SHA, last commit date, licence | verse-lab/Lentil @ `1ddf1fc16138371cae6687db2698770533c4058a`, 2026-11-06 01:09:54 +0800, Apache-2.0. Public history is a single commit. |
| Lean toolchain (or other tool and version) | `leanprover/lean4:v4.35.0`; no Mathlib (`RelationalRanking/Rules.lean` re-proves needed list lemmas in a "TheoremsFromMathlib" section). |
| Maturity | Research library in active use as a Veil dependency (Veil pins the same commit `1ddf1fc`). Ships an "appendix for AI agents" with instructions "when an AI agent writes Lentil proofs" (`docs/Lentil-agent-appendix.md:1-4`), indicating the authors expect AI-written proofs. AI provenance: `agent-config-present`. This does not show that the library code is AI-generated, and the single squashed commit prevents commit-pattern analysis. Evidence level: `static-inspected`. |

## 2. Claim as stated by the authors (verbatim)
- README (S1): "Lentil is a proof infrastructure for linear temporal logic in Lean 4. It formalizes the semantics of LTL, provides rules and a dedicated proof mode to conduct proofs in LTL as an object logic." and "The definitions of the embedding are initially ported from [`coq-tla`]... The design of the proof mode is partly inspired by [Iris Proof Mode]."
- S3: "Veil's temporal reasoning is built on Lentil, a small Lean library formalizing TLA (itself inspired by the Rocq library coq-tla)." and "The purpose of this post is to show that liveness is within comfortable reach of a Lean-based verifier, not just safety."

## 3. What is actually formalised
Semantics (`Lentil/Basic.lean`, S2):
```lean
def exec (σ : Type u) := Nat → σ                                  -- line 24
def pred (σ : Type u) := exec σ → Nat → Prop                       -- line 25
def always {α : Type u} (p : pred α) : pred α := fun e i => ∀ k, p e (i + k)      -- line 65
def eventually {α : Type u} (p : pred α) : pred α := fun e i => ∃ k, p e (i + k)  -- line 66
def later {α : Type u} (p : pred α) : pred α := fun e i => p e (i + 1)            -- line 67
def enabled {α : Type u} (a : action α) (s : α) : Prop := ∃ s', a s s'            -- line 88
def tl_until {α : Type u} (p q : pred α) : pred α :=
  fun e i => ∃ k, q e (i + k) ∧ ∀ l, l < k → p e (i + l)                          -- line 118
def Lentil.leads_to {α : Type u} (p q : Lentil.pred α) : Lentil.pred α := [tlfml| □ (p → ◇ q) ]            -- line 199
def Lentil.weak_fairness {α : Type u} (a : action α) : pred α := [tlfml| □ ((□ (Enabled a)) → ◇ ⟨a⟩)]      -- line 201
def Lentil.strong_fairness {α : Type u} (a : action α) : pred α := [tlfml| □ ((□ ◇ (Enabled a)) → ◇ ⟨a⟩)] -- line 202
```
Behaviours are total functions `Nat → σ` (infinite traces); there is no stuttering-insensitive subscript `⟨A⟩_v` built in. S3 footnote 6: Lentil's `⟨ ... ⟩` "is just the lifting of definitions to temporal formulas, with no subscript and no non-stuttering condition."

Proof rules (examples, verbatim):
- `Lentil/Rules/WF.lean:46`:
  ```lean
  theorem wf1 {p q : pred σ} (next a : action σ) :
    ((p ∧ ⟨next⟩ ⇒ ◯ p ∨ ◯ q) ∧
     (p ∧ ⟨next⟩ ∧ ⟨a⟩ ⇒ ◯ q) ∧
     (p ⇒ Enabled a ∨ q) ∧
     (□ ⟨next⟩ ∧ 𝒲ℱ a)) |-tl- (p ↝ q)
  ```
- `Lentil/Rules/WF.lean:80`:
  ```lean
  theorem wf1_original {p q : pred σ} (next a : action σ) :
    ((p ∧ ⟨next⟩ ⇒ ◯ (p ∨ q)) ∧
     ((p ∧ ⟨next⟩ ∧ ⟨a⟩ ⇒ ◯ q)) ∧
     ((p ⇒ Enabled a))) |-tl- (□ ⟨next⟩ ∧ 𝒲ℱ a → p ↝ q)
  ```
- `Lentil/Rules/SF.lean:47` `sf1`, `:78` `sf1_original`, `:35` `sf_implies_wf : (𝒮ℱ a) |-tl- (𝒲ℱ a)`.
- `Lentil/Rules/LeadsTo.lean:36` `leads_to_trans`, `:44` `leads_to_combine`, `:29` `leads_to_conseq`, `:20` `leads_to_exists`.
- `Lentil/Rules/Basic.lean:182` `always_induction : □ p =tl= (p ∧ □ (p → ◯ p))`.
- 303 `theorem`/`lemma` declarations in `Lentil/` and `LentilApplications/`.

Proof mode: tactics prefixed with `t` (`tstart`, `tintro`, `tapply`, `tsuffices`, `tclear`, `trevert`, `trewrite`, `tsplit_ands`, `tmonotone`, `trcases`, `thave`, `tspecialize`) in `Lentil/ProofMode/Tactics/*.lean`. `Lentil/Tactics/FiniteWindow.lean` (840 lines) implements "a finite-window reduction for local LTL obligations", where "A `FiniteWindow p n` contains a curried finite predicate `core` over exactly `n` states, together with a theorem saying that evaluating `p` on a trace is equivalent to evaluating `core` on the first `n` states of that trace." `Lentil/ProofMode/Tactics/CoalesceToPTL.lean` reduces to propositional temporal logic.

Applications: `LentilApplications/RelationalRanking/Rules.lean` (730 lines), titled "Relational Ranking Technique for Proving Liveness Properties", says "Let's validate the rules from [this paper](https://link.springer.com/chapter/10.1007/978-3-031-65627-9_13)!" and proves `rule6`..`rule11` (lines 503-695), e.g. `theorem rule7 : (C0 ∧ D1 ∧ D2 ∧ D3 ∧ D4) |-tl- (p ↝ q)` (line 654) and `theorem rule6 : (C0 ∧ C1 ∧ C2 ∧ C3) |-tl- (□ ◇ r → (p ↝ q))` (line 695). This is a mechanised check of published proof rules, not a protocol proof.

Protocol use (via Veil, REF-L04): `veil/Examples/Liveness/TLA/Consensus.lean:47` proves `𝒲ℱ choose → ◇ ⌜ someChosen ⌝` for a one-step consensus abstraction with `tapply wf1_original`; S3 gives the complete script and notes "for now we model fairness by stating it directly as a premise of the temporal property."

## 3a. Predicate semantics
The headline theorems are the proof rules, such as `wf1` (`Lentil/Rules/WF.lean:46-50`, S2 @ 1ddf1fc):
```lean
theorem wf1 {p q : pred σ} (next a : action σ) :
  ((p ∧ ⟨next⟩ ⇒ ◯ p ∨ ◯ q) ∧
   (p ∧ ⟨next⟩ ∧ ⟨a⟩ ⇒ ◯ q) ∧
   (p ⇒ Enabled a ∨ q) ∧
   (□ ⟨next⟩ ∧ 𝒲ℱ a)) |-tl- (p ↝ q)
```
Definitions of the conclusion predicate (`Lentil/Basic.lean:24-25,65-66,77-80,199-201`):
```lean
def exec (σ : Type u) := Nat → σ
def pred (σ : Type u) := exec σ → Nat → Prop
def always {α : Type u} (p : pred α) : pred α := fun e i => ∀ k, p e (i + k)
def eventually {α : Type u} (p : pred α) : pred α := fun e i => ∃ k, p e (i + k)
def pred_implies {α : Type u} (p q : pred α) : Prop := ∀ (σ : exec α) i, p σ i → q σ i
def valid {α : Type u} (p : pred α) : Prop := ∀ (σ : exec α) i, p σ i
def Lentil.leads_to {α : Type u} (p q : Lentil.pred α) : Lentil.pred α := [tlfml| □ (p → ◇ q) ]
def Lentil.weak_fairness {α : Type u} (a : action α) : pred α := [tlfml| □ ((□ (Enabled a)) → ◇ ⟨a⟩)]
```
`|-tl-` denotes `pred_implies` (notation, `Lentil/Basic.lean:212-213`).

Plain language. `wf1` says the following for every infinite state sequence and every position in it. If every `next` step preserves `p` or reaches `q`; if every `a` step from `p` reaches `q`; if `p` implies that `a` is enabled or `q` holds; and if the sequence is a `next` behaviour that is weakly fair to `a`, then from that position `p` always leads to `q`. What is guaranteed: the rule is sound over the shallow embedding, that is, over Lean functions `Nat → σ`. What is not guaranteed: that any concrete system satisfies the premises, in particular fairness, which is a hypothesis and never derived. Executions are infinite by construction, so terminating systems need stuttering. No link to running code exists.

## 4. Property classes covered
| Class | Status | Evidence |
|---|---|---|
| Functional correctness | no | Library only. |
| Safety invariant | partial | `always_induction` and `□` reasoning; Veil safety uses its own VC pipeline. |
| Liveness | proved (as library rules) / partial (as applied) | WF1, SF1, leads-to rules proved sound; applied to small Veil examples only (S2, S3). |
| Refinement | no | No refinement mappings or stuttering-invariance theorems found; S3 calls refinement "orthogonal". |
| Symbolic security | no | |
| Computational security | no | |
| Parsing | no | |
| Constant-time | no | |
| Other | proved | Mechanised validation of relational-ranking liveness rules (`LentilApplications`). |

## 5. Link to implementation code
None. Lentil reasons about abstract traces `Nat → σ`; it is consumed by Veil models, which are themselves not linked to external implementations (REF-L04 section 5).

## 6. Trusted computing base and soundness hygiene
Keyword counts across 12,004 Lean lines (`rg -w`, incl. comments): `sorry` 1, `admit` 0, `axiom` 0, `native_decide` 0, `implemented_by` 0, `extern` 0, `opaque` 7, `partial` 19. The single `sorry` hit is in a comment (`Lentil/ProofMode/Tactics/Rename.lean:14`: "elaborated to `sorry` (Lean 4.32)"). `partial`/`opaque` hits are in meta-level tactic code. TCB: Lean kernel plus standard axioms; tactic code is untrusted (its output is kernel-checked). In Veil, the first-order obligations left after `tmonotone` are discharged by `veil_solve_temporal`, which can use SMT; whether those are trusted depends on Veil's `veil.smt.trust` option (REF-L04 section 6). The Consensus example sets it to `false` and audits axioms with `Lean.collectAxioms` (`veil/Examples/Liveness/TLA/Consensus.lean:39,80`).

## 7. Scale, effort, automation
- LOC at commit: 12,004 total (`Lentil/` 7,423; `LentilApplications/` 852; `LentilTest/` 3,683). Docs: 1,308 lines (`Lentil-reference.md` 1,036).
- Largest files: `Lentil/Tactics/FiniteWindow.lean` 840, `LentilApplications/RelationalRanking/Rules.lean` 730, `Lentil/Rules/Basic.lean` 546.
- Automation: finite-window reduction and PTL coalescing; otherwise interactive. No reported effort figures.
- Not built here (section 10).

## 8. Limitations stated by the authors (verbatim)
- S3: "for now we model fairness by stating it directly as a premise of the temporal property."
- S3: "We begin here with the smallest example we could find—deliberately so, to keep the protocol out of the way"; "In the next posts in this series, we will keep this machinery fixed and grow the protocol".
- S3 footnote 3: "Refinement is an important topic in TLA+, but it is orthogonal to what we focus on in this post, so we will not discuss it further here."
- S1 README: "Lentil is built against the Lean version pinned in `lean-toolchain`."

## 9. Independent evidence
No independent reviews, adoption beyond Veil, or bugs found were located. Related independent Lean temporal-logic work exists (LeanLTL, ITP 2025; leslie) but was not read in full (section 15).

## 10. Reproducibility
Not built: sandbox disk limit; queued for CI reproducibility job. The attempt `cd /tmp/lentil && timeout 1500 lake build` exited 1 after 73 s because elan failed to extract `leanprover/lean4:v4.35.0`: "No space left on device (os error 28)" on the shared sandbox; it was not retried. Exact command for CI:
```
git clone --depth 1 https://github.com/verse-lab/Lentil && cd Lentil && git checkout 1ddf1fc16138371cae6687db2698770533c4058a \
  && curl https://raw.githubusercontent.com/leanprover/elan/master/elan-init.sh -sSf | sh -s -- -y --default-toolchain none \
  && timeout 1500 lake build
```

## 11. Generalisation
Pattern: shallow embedding of TLA-style temporal logic over `Nat → σ` traces, with Lamport's fairness rules as Lean theorems and an Iris-style proof mode that reduces temporal goals to one-step first-order VCs. Any transition system expressible in Lean (not only Veil modules) can be given a `pred σ` specification. To use it for an arbitrary codebase one would still need a model of the code as `σ` and `action σ` and a refinement argument from code to that model; Lentil provides neither. Preconditions: fairness assumptions stated explicitly as hypotheses; no built-in stuttering invariance.

## 12. Bearing on sub-claims (facts only, no verdict)
| Sub-claim | Fact from this reference | Source ID |
|---|---|---|
| H1 | LTL/TLA operators, weak/strong fairness and leads-to are defined in Lean (`Lentil/Basic.lean:65-202`). | S2 |
| H2 | WF1/SF1 and leads-to rules are proved; applied liveness proofs found are for a one-step consensus model and one FloodSet property; relational-ranking rules from a published paper are mechanised. | S2, S3 |
| H3 | No implementation link. | S2 |
| H4 | Library is language-agnostic at the model level but has no front end for any programming language. | S2 |
| H5 | Not addressed. | |
| D | "Prove" here means kernel-checked temporal sequents `|-tl-` over all infinite traces; S3 contrasts this with TLAPS. | S2, S3 |

## 13. Conflicts
None found between sources. Note: S3 says Lentil is "a small Lean library", while the repo is 12,004 lines including tests; this is a characterisation difference, not a factual conflict.

## 14. Sources
| ID | Title | URL | P/S | Depth | Summary |
|---|---|---|---|---|---|
| S1 | verse-lab/Lentil README and docs | https://github.com/verse-lab/Lentil | P | [FULL] README.md and docs/ | States purpose, coq-tla origin, Iris-inspired proof mode; docs include an AI-agent appendix. |
| S2 | verse-lab/Lentil source @ 1ddf1fc | https://github.com/verse-lab/Lentil | P | [CODE] @ 1ddf1fc | Source of all definitions, theorem statements, counts. |
| S3 | Liveness Proofs in Veil, Part I: The First Step (Zhao, Sergey, 2026-06-24) | https://proofsandintuitions.net/2026/06/24/liveness-proofs-in-veil-part-1/ | P | [FULL] | End-to-end WF1 liveness proof in Veil using Lentil; states fairness-as-premise and refinement scope limits. |
| S4 | veil `Examples/Liveness/TLA/Consensus.lean` @ 5744772 | https://github.com/verse-lab/veil | P | [CODE] @ 5744772 | Shows Lentil used in Veil, `veil.smt.trust false`, axiom audit. |

## 15. Unverified leads (excluded from evidence)
- LeanLTL: A Unifying Framework for Linear Temporal Logics in Lean (ITP 2025), https://drops.dagstuhl.de/entities/document/10.4230/LIPIcs.ITP.2025.37 (search result only).
- The relational-ranking paper linked from `RelationalRanking/Rules.lean`: https://link.springer.com/chapter/10.1007/978-3-031-65627-9_13 (not read).
- coq-tla, https://github.com/tchajed/coq-tla (origin of the embedding; not inspected).
- rupakm/leslie, https://github.com/rupakm/leslie (another Lean temporal/distributed framework; covered by REF-L19).
