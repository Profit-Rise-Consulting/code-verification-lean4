# REF-L17: raft-lean-squad (Lean Squad applied to raft-rs)

## 1. Identity
| Field | Value |
|---|---|
| Subject (protocol or algorithm) | Raft consensus as implemented in the Rust `raft` crate (fork of `tikv/raft-rs`): quorum arithmetic, log management (`find_conflict`, `maybe_append`, `maybe_commit`, `RaftLog::append`), flow control (`Inflights`), progress tracking, joint configuration, read-only, election vote granting, plus an abstract cluster-level safety model |
| Spec it targets | No RFC. Informal specs were written by the agent from the Rust code (`formal-verification/specs/*_informal.md`, 25 files); the cluster-level property is Raft "state machine safety" (no two nodes apply different entries at one index) |
| Organisation / authors | Don Syme (GitHub Next). All Lean, specs and reports were produced by the "Lean Squad" GitHub Agentic Workflow (Copilot CLI with Claude Opus 4.6 / Sonnet 4.6) [S2] |
| Primary URLs | https://github.com/dsyme/raft-lean-squad ; blog https://dsyme.net/2026/04/20/lean-squad-automated-software-verification-with-near-zero-human-labour/ |
| Repo, commit SHA, last commit date, licence | `dsyme/raft-lean-squad` @ `ab7a184fbdfe22ac3b894e92791fb9f324e2ccbc` ("update workflows"), 2026-06-03 15:38 +0100, Apache-2.0 (inherited from raft-rs) |
| Lean toolchain (or other tool and version) | `formal-verification/lean/lean-toolchain`: `leanprover/lean4:v4.28.0`; stdlib only (`lake-manifest.json` has no packages). REPORT.md and CRITIQUE.md say "Lean 4.30.0-rc2" (see Conflicts) |
| Maturity | AI-generated research experiment. Determined from: README/REPORT banner "Lean Squad — automated formal verification"; blog "Every definition and every statement of every theorem is AI written"; `paper/paper.tex` lists authors "Don Syme", "LeanSquad", "Copilot CLI", "Claude Sonnet 4.6"; workflows `.github/workflows/lean-squad.md` and `lean-squad.lock.yml`. The human contribution was "three GitHub nudge comments" and merging PRs [S2] |

## 2. Claim as stated by the authors (verbatim)
REPORT.md (dated 2026-04-28):
> "**Status**: ✅ **COMPLETE** — 716 theorems, 79 Lean files, **0 `sorry`**, machine-checked by Lean 4.30.0-rc2 (stdlib only). Top-level safety theorem `fullProtocolStep_safe` (EL7) proved with **all 5 `RaftReachable.step` hypotheses fully discharged** — no abstract axioms remain."

> "Proved **unconditional end-to-end Raft cluster safety** (`fullProtocolStep_safe`, EL7): given any reachable cluster state, an election epoch, and a valid AppendEntries step, the resulting state is safe"

> "Validated **28 correspondence targets** via 625 `#guard` tests and Rust test functions"

Blog (2026-04-20), describing an earlier snapshot:
> "The top-level formal result is raftReachable_safe: for any cluster state reachable via valid Raft transitions from an empty initial state, no two nodes ever apply different log entries at the same committed index. That's qualified state-machine safety for Raft, machine-checked by Lean 4 with zero "sorry"."

> "I never selected a target, wrote a specification, suggested a proof strategy, or debugged a failing proof. I didn't need to know Lean."

## 3. What is actually formalised
Cluster state is an abstract snapshot, not the Rust `Raft` struct. `FVSquad/RaftSafety.lean:276-290`:
```lean
structure ClusterState (E : Type) where
  voters    : List Nat
  logs      : VoterLogs E
  committed : Nat → Nat   -- per-node committed index

def isClusterSafe [DecidableEq E] (cs : ClusterState E) : Prop :=
  ∀ v1 v2 : Nat, ∀ k : Nat, ∀ e1 e2 : E,
    nodeHasApplied cs v1 k e1 → nodeHasApplied cs v2 k e2 → e1 = e2
```
Reachability is an inductive predicate whose step constructor carries the protocol validity conditions as hypotheses. `FVSquad/RaftTrace.lean:110-142`:
```lean
inductive RaftReachable [DecidableEq E] : ClusterState E → Prop where
  | init (voters : List Nat) :
      RaftReachable (initialCluster voters)
  | step {cs cs' : ClusterState E}
      (hreach : RaftReachable cs)
      (hvoters : cs.voters = cs'.voters)
      (v : Nat)
      (hlogs' : ∀ w, w ≠ v → cs'.logs w = cs.logs w)
      (hno_overwrite : ∀ k, cs.committed v ≥ k → cs'.logs v k = cs.logs v k)
      (hqc_preserved : ∀ k e, isQuorumCommitted cs.voters cs.logs k e →
          isQuorumCommitted cs'.voters cs'.logs k e)
      (hcommitted_mono : ∀ w, cs'.committed w ≥ cs.committed w)
      (hnew_cert : ∀ w k e, cs'.committed w ≥ k → cs.committed w < k →
          cs'.logs w k = some e →
          isQuorumCommitted cs'.voters cs'.logs k e) :
      RaftReachable cs'
```
`FVSquad/RaftTrace.lean:228-233`:
```lean
theorem raftReachable_safe [DecidableEq E]
    (hd : Nat) (tl : List Nat) (cs : ClusterState E)
    (hvoters : cs.voters = hd :: tl)
    (hreach : RaftReachable cs) :
    isClusterSafe cs :=
  raft_end_to_end_safety_full hd tl cs hvoters (raftReachable_cci cs hreach)
```
Headline theorem `FVSquad/ElectionLifecycle.lean:248-258`:
```lean
theorem fullProtocolStep_safe {E : Type} [DecidableEq E]
    {lead hd : Nat} {tl : List Nat}
    {cs_pre cs_post cs_next : ClusterState E}
    (hreach_pre : RaftReachable cs_pre)
    (epoch : ElectionEpoch E lead hd tl cs_pre cs_post)
    {v_next : Nat} {msg_next : AppendEntriesMsg E}
    (hstep : ValidAEStep E cs_post lead v_next msg_next cs_next)
    (hvoters_next : cs_next.voters = hd :: tl) :
    isClusterSafe cs_next
```
The arguments `epoch` and `hstep` are structures whose fields are themselves protocol conditions. `ElectionEpoch` (`ElectionLifecycle.lean:90-128`) bundles a vote record, `hwin : wonInTerm ...`, `hconsist : VoteRecordConsistency ...`, a `BroadcastSeq`, and field constraints including
```lean
  /-- Leader is not in the voter list (leader doesn't send AE to itself). -/
  hlead_not_in : lead ∉ (hd :: tl)
```
`ValidAEStep` (`FVSquad/ConcreteProtocolStep.lean:145-180`) is a `Prop`-valued structure that includes, among its fields, the `hqc_preserved` condition verbatim:
```lean
  hqc_preserved : ∀ k e, isQuorumCommitted cs.voters cs.logs k e →
      isQuorumCommitted cs'.voters cs'.logs k e
```
So a caller of `fullProtocolStep_safe` must supply objects that contain these conditions; the theorem does not derive them from executions of the Rust code or from a message-level network model.

Function-level components (e.g. `committedIndex_safety`, `committedIndex_maximality`, `limitSize_maximality`, `voteResult_Won_iff`) are Lean re-implementations of individual Rust functions over `Nat`/`List`, with theorems about those Lean functions (CRITIQUE.md Layers 1-2). Environment: no network, message loss, crashes, timing or concurrency is modelled; numbers are `Nat`; voter sets are `List Nat` rather than `HashSet<u64>` (CRITIQUE "Concern 1", "Concern 3").

## 4. Property classes covered
| Class | Status | Evidence |
|---|---|---|
| Functional correctness | partial | Per-function specs on Lean models of ~28 Rust functions (e.g. `limitSize_maximality`, `committedIndex_maximality`), CRITIQUE.md Layers 1-2 |
| Safety invariant | partial | `raftReachable_safe` / `fullProtocolStep_safe` prove `isClusterSafe` over the abstract `RaftReachable` relation, conditional on step-structure fields (Section 3) |
| Liveness | no | Not attempted; no temporal operators in the repo |
| Refinement | no | No refinement theorem between the Rust state machine and `ClusterState` |
| Symbolic / computational security, parsing, constant-time | no | Out of scope |
| Other: compile-time conformance tests | yes | 696 `#guard` lines across FVSquad (642 in correspondence files per CRITIQUE) |

## 5. Link to implementation code
Mechanism: manual (agent) transcription plus differential example testing. The blog states: "In Lean Squad, it can do this either by “Path A” – using the automated tool Aeneas to translate Rust code to Lean, or “Path B” – “Path A” – writing new Lean definitions that “correspond” to the implementation code." (the "Path B" – "Path A" duplication is in the original) and "Unfortunately I was unable to get Aeneas to successfully process the codebases from the first two examples, so Path B was chosen for both." [S2]. A `Charon.toml` is present at the repo root, but no Aeneas-generated Lean exists in `formal-verification/lean`.

Correspondence: shared fixtures in `formal-verification/tests/<target>/cases.json` are mirrored (a) as Lean `#guard` checks, e.g. `FVSquad/FindConflictCorrespondence.lean` ("each `#guard` assertion runs the Lean model on a concrete test case and verifies the result at compile time"), and (b) as Rust unit tests, e.g. `src/raft_log.rs:2118 fn test_find_conflict_correspondence()`. 39 occurrences of "correspondence" across `src/`. Guarantee: agreement on finitely many hand-picked inputs (17 cases for `find_conflict`); CRITIQUE: "`#guard` tests are bounded to small inputs". Documented divergences include `JointCommittedIndex` returning 0 vs Rust `u64::MAX` on empty configs, an `UncommittedState` noLimit path divergence, and only the non-group-commit path being modelled.

## 6. Trusted computing base and soundness hygiene
Static count at `ab7a184` over 80 `.lean` files, with comments and strings stripped: `sorry` 0, `admit` 0, `axiom` 0, `native_decide` 2, `implemented_by` 0, `extern` 0, `opaque` 0, `partial def` 0. (A raw word grep gives sorry=29, axiom=2, all inside doc comments.) The 2 `native_decide` uses: `FVSquad/ConfigurationInvariantsCorrespondence.lean:81` (an `example`) and `FVSquad/ConfigValidate.lean:184 theorem defaultCfg_valid`. They add the Lean compiler to the TCB for those two facts.
Unverified assumptions: the faithfulness of each hand-written Lean model to its Rust function (checked only by `#guard` examples and AI-written CORRESPONDENCE.md, 2,710 lines); the fields of `ElectionEpoch` and `ValidAEStep`; `HLogConsistency` (CRITIQUE: "still taken as a conditional hypothesis in `LeaderCompleteness.lean` (LC7/LC8)").

## 7. Scale, effort, automation
- Lean: 80 files, 21,355 lines; 769 `theorem`/`lemma` declarations (matches CRITIQUE's 769; REPORT says 716); 696 `#guard` lines.
- Process: 129 automated runs (REPORT); blog for the earlier 34-run, ~3-week phase: "total token cost of this activity was (very approximately) $240 total (this is estimating $7 per run × 34 runs, Claude Opus 4.6 pricing, though in practice the much cheaper Claude Sonnet 4.6 may be been [used]...)". Human input: three nudge comments.
- Automation: LLM coding agent via GitHub Agentic Workflows, scheduled every 8 hours, phase-weighted task selection over ten task types (Research, Informal Spec, Formal Spec, Implementation, Proof, Correspondence Review, Critique, CI, Report, Paper) [S2].
- Build time: not measured (not built, Section 10).

## 8. Limitations stated by the authors (verbatim)
Blog [S2]:
> "**The definitions are AI written**. Every definition and every statement of every theorem is AI written. There is no guarantee that these are "good" definitions."
> "If Path B is chosen, it is possible the AI will "make up some garbage implementation that's easy to prove something about"."
> "**The theorems may have assumptions.** In raft-rs, concrete message types and leader election are abstracted into the "RaftReachable" step hypotheses."
> "**Most theorems are glorified testing**. The majority of theorems proven are very directly obvious from the code."
> "In retrospect the automated "critique" step I implemented was not strong enough, and the process allowed a form of "pseudo-completion" where key assumptions were actually present in the preconditions to the headline theorem."

CRITIQUE.md (2026-04-28):
> "The Lean model covers only `use_group_commit = false`. The group-commit path uses a different algorithm and is completely unverified."
> "All numeric types are `Nat` in Lean. Overflow scenarios are not covered."
> "`HLogConsistency` ... is still taken as a conditional hypothesis in `LeaderCompleteness.lean` (LC7/LC8)."

## 9. Independent evidence
No bugs found in raft-rs. CRITIQUE: "**No bugs found** in any modelled Rust function." Self-reported findings are formulation issues: an over-strict `VotersLearnersDisjoint` predicate, and RSS3/RSS4 "initially stated as universally quantified properties that are provably false". No independent external review was found. The author's own blog is the main critique: it identifies the "pseudo-completion" problem. The same workflow on other repos is reported in the blog to have found a Rust `Ord` contract violation in quiche ("both `a.cmp(b) = Greater` and `b.cmp(a) = Greater` hold simultaneously") and, for PX4, "259 Theorems, Two Bugs". Those are outside this dossier.

## 10. Reproducibility
Not built. A stdlib-only build was started (`lake build` in `formal-verification/lean`), but I stopped it while it was still downloading the v4.28.0 toolchain. Shared sandbox disk was at 96-97% (under 1 GB free), and the orchestrator then set a no-build policy for this phase. The repo has CI workflow `.github/workflows/lean-ci.yml`, and the blog states "`lake build` passes in CI on every PR". That CI result was not independently observed.

## 11. Generalisation
Pattern: an AI agent writes a hand-transcribed Lean model of each implementation function, proves theorems about that model, and ties it to the source with shared example fixtures checked by both Lean `#guard` and native unit tests. A separate abstract protocol model is related to the function models only by documentation. This pattern can be applied to any language that can run the same fixtures (here Rust; the blog also reports C++ and Rust/QUIC). Requirements for an arbitrary codebase: an agent workflow with Lean in CI, fixture plumbing in the target language's test harness, and human or agent review of the CORRESPONDENCE document. Specific to this case: raft-rs is a pure consensus module with no I/O ("includes the core Consensus Module only", README), which makes function-level models practical. Aeneas extraction (Path A) failed here.

## 12. Bearing on sub-claims (facts only, no verdict)
| Sub-claim | Fact from this reference | Source ID |
|---|---|---|
| H1 | Raft cluster state, quorum certification and a reachability relation are expressed in Lean stdlib. Messages, network and time are not modelled | S1 |
| H2 | 769 theorems with 0 sorry and 0 axiom. The headline safety theorem takes `ElectionEpoch`/`ValidAEStep` structures that contain protocol conditions (including `hqc_preserved`) as fields | S1, S2 |
| H3 | The link to Rust is hand transcription plus shared-fixture `#guard`/`cargo test` example agreement. Aeneas extraction was attempted and failed | S1, S2 |
| H4 | The same agent workflow was reportedly applied to Rust (raft-rs, quiche) and C++ (PX4) | S2 |
| H5 | Function-level Lean models were checked against Rust by testing, independently of the protocol-level proof chain (25 of 50 proof files are standalone, REPORT) | S1 |
| D | The authors use "machine-checked", "proved" and "verified". The blog calls most theorems "glorified testing". The `#guard` checks are called "correspondence validation" | S1, S2 |

## 13. Conflicts
- REPORT.md: "all 5 `RaftReachable.step` hypotheses fully discharged — no abstract axioms remain". CRITIQUE.md (same date): "`hqc_preserved` | **Conditionally discharged** | CPS13 via `CandidateLogCovers`", and "`HLogConsistency` ... is still taken as a conditional hypothesis". Code: `ValidAEStep` contains `hqc_preserved` as a structure field (`ConcreteProtocolStep.lean`).
- Theorem count: REPORT 716 theorems vs CRITIQUE 769 theorem declarations. My static count is 769 `theorem`/`lemma` lines. Blog: 443 theorems across 23 files (an earlier snapshot).
- Toolchain: REPORT/CRITIQUE "Lean 4.30.0-rc2" vs `lean-toolchain` file `leanprover/lean4:v4.28.0`.
- `#guard` count: REPORT 625 vs CRITIQUE 642 (+54 in proof files). My grep count is 696 total.
- Blog claim "zero sorry" for `raftReachable_safe` vs the blog's own quotation of an interim critique: "`raftReachable_safe` (RT2) is conditionally correct only: it proves safety *given* the hypotheses, but the hypotheses themselves are axioms rather than proved facts." Both are in S2 and describe different times.

## 14. Sources
| ID | Title | URL | P/S | Depth | Summary |
|---|---|---|---|---|---|
| S1 | dsyme/raft-lean-squad repo @ ab7a184 (FVSquad Lean, REPORT.md, CRITIQUE.md, CORRESPONDENCE.md, src/ tests) | https://github.com/dsyme/raft-lean-squad | P | CODE | Fork of raft-rs with an AI-generated `formal-verification/` tree: 80 Lean files, 769 theorems, 0 sorry, 2 native_decide. Abstract Raft safety is proved over a hypothesis-carrying reachability relation, with fixture-based Rust correspondence |
| S2 | Don Syme, "Lean Squad: Exploring Automated Software Verification with Near-Zero Human Labour" (2026-04-20) | https://dsyme.net/2026/04/20/lean-squad-automated-software-verification-with-near-zero-human-labour/ | P | FULL | Describes the agentic workflow, cost (~$240 for 34 runs), the three nudges, the Aeneas failure, the "pseudo-completion" problem and explicit limitations |

Access date for all: 2026-11-07.

## 15. Unverified leads (excluded from evidence)
- `formal-verification/paper/paper.tex` / `paper.pdf` (draft "Truly Automated Formal Specification and Verification with Near-Zero Human Labour"): only the title/author block was read.
- Lean Squad workflow definition at `githubnext/agentics/lean-squad` was not read.
- quiche and PX4 Lean Squad forks (blog-reported bugs) were not inspected.
