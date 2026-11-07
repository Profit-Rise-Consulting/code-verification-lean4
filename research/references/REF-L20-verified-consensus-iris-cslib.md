# REF-L20: verified-consensus (fradamt), plus iris-lean and the CSLib distributed-consensus status

This dossier covers three subjects. verified-consensus is primary. iris-lean is covered as the Lean concurrent-separation-logic infrastructure that distributed-systems proofs in the Iris tradition would build on. For CSLib, the task asked about "any cslib Raft port": none was found, and the distributed-systems content CSLib does have (FLP) is recorded instead.

## 1. Identity
| Field | verified-consensus | iris-lean | CSLib (FLP) |
|---|---|---|---|
| Subject | Executable Lean 4 model of a "decoupled consensus protocol" (Section 7 of an unpublished paper): cumulative store, fork choice, grades, validator duties, finality gadget, tick handler. Proves safety, accountable safety, liveness and asynchrony-resilience claims | Lean 4 port of Iris higher-order concurrent separation logic: MoSeL proof mode, `IProp` model, HeapLang, program logic (weakest preconditions, adequacy) | Völzer's proof of FLP impossibility: no 1-fault-tolerant asynchronous consensus |
| Spec it targets | `consensus.tex` (not public). Section 7 extracted to `docs/PROTOCOL.md` | Iris-Rocq (https://gitlab.mpi-sws.org/iris/iris/) | Fischer-Lynch-Paterson 1985; Völzer 2004 |
| Authors | Francesco D'Amato (`CITATION.cff`) | leanprover-community (Iris-Lean contributors) | Ching-Tsun Chou (file headers) |
| Primary URLs | https://github.com/fradamt/verified-consensus | https://github.com/leanprover-community/iris-lean | https://github.com/leanprover/cslib (`Cslib/Computability/Distributed/FLP/`) |
| Repo, commit SHA, last commit date, licence | `d379754762ccd69cec2fcaf94d18481a0600ef78`, 2026-10-12, CC0-1.0 | `8293092f09bd32ec60640f4267e7b7cd98bfe024`, 2026-11-05, Apache-2.0 | `76b7943d0b8d4673b207b82fea86610e7f74d625`, 2026-11-05, Apache-2.0 |
| Lean toolchain | `leanprover/lean4:v4.34.1` + Mathlib | `v4.35.0` (Iris, IrisMath) | `v4.36.0-rc2` + Mathlib |
| Maturity | Research, review-oriented, AI-assisted. Determined from: `CITATION.cff` (date-released 2026-09-23); `docs/AI_AUDIT.md` "Brief for an AI reviewer"; commit trailers in the last 100 commits include `Co-Authored-By: Claude Opus 5.5` (1) and two `Claude-Session:` links | Community library, active (dependabot commit 2026-11-05; 247 stars) | Community library; FLP merged via PRs #556, #619, #718, #755, #765 |

## 2. Claim as stated by the authors (verbatim)
verified-consensus README:
> "A Lean 4 model of the decoupled consensus protocol (Section 7 of the paper) and a machine-checked proof of its consensus guarantees."

> "One theorem proves all of them. The proof has no `sorry` and no project-specific axiom."

> "The results assume well-formed executions with unforgeable honest signatures and collision-free roots, bounded delivery after GST, and honest committee majorities. Liveness also needs participation and proposer recurrence."

> "Only two libraries must be trusted, and both hold definitions only: `DecoupledConsensusModel` (29 files, about 5,700 lines) and `DecoupledConsensusStatements` (12 files, about 1,150 lines). The kernel checks the rest"

iris-lean `readme.md`:
> "Lean 4 port of *Iris*, a higher-order concurrent separation logic framework."

CSLib FLP README:
> "This directory contains a formalization of Völzer's proof [Volzer2004] of the famous result in distributed computing, first proved by Fischer, Lynch and Paterson [FLP1985], that asynchronous distributed consensus is impossible in the presence of even a single crash fault."

## 3. What is actually formalised
verified-consensus. The review theorem (`DecoupledConsensusProofs/ReviewTheorem.lean:288`):
```lean
theorem concreteConsensus (S : Setup V) : Statements.Instantiation.Consensus S := by
```
`Consensus` is a 13-field `Prop` structure (`DecoupledConsensusStatements/Generic/Claims.lean:48`). Each field quantifies over runs satisfying a named regime, for example:
```lean
  finalizedSafe : ∀ rho, AccountableRegime E I rho → AgreeFrom P rho I.finalized 0
  /-- Sleepy regime. -/
  available : ∀ rho t₀, SleepyRegime P E I C rho t₀ → AvailableAt P I C rho t₀
  confirmedLive : ∀ rho t₀ gap, LiveSleepyRegime P E I C rho t₀ gap →
    LiveFrom P I rho I.confirmed t₀ (C.growthDelay gap)
```
Environment model (README "Premise ledger"): runs are "sorted lists of tick and delivery events at honest nodes, folded into per-node states". `ExecutionValid` = `ScheduleWellFormed` + `DeliveryWellFormed` + `UnforgeableSignatures` + root collision freedom. Partial synchrony with GST. Byzantine nodes are modelled through which objects reach honest nodes (AI_AUDIT dimension 1). Cryptography is idealised as premises. The model is an executable state machine written in Lean (`DecoupledConsensusModel/Protocol/`), not a link to a client implementation.

iris-lean. A separation-logic framework (`Iris/Iris/BI`, `ProgramLogic/WeakestPre.lean`, `Adequacy.lean`, `HeapLang/`). It contains no distributed-systems or network-semantics development: a search for `raft|paxos|consensus|network|socket` in `.lean` files returned nothing. There is no Lean counterpart of Aneris/Trillium in the repo.

CSLib FLP (`Cslib/Computability/Distributed/FLP/Impossibility.lean:154`):
```lean
theorem Consensus.one_not_exists [Fintype P] (hc : 2 ≤ card P) :
    ¬ ∃ a : Algorithm P M S, a.Consensus 1 := by
```
with a generalisation `Consensus.ge_one_not_exists` for any `f ≥ 1`. `ZeroConsensus.lean` gives a simple algorithm that achieves consensus with no faults.

CSLib Raft port: none found. In the clone at `76b7943` a search for `raft` (word match) in Lean sources finds nothing. `gh search prs --repo leanprover/cslib raft` and `gh search issues ... raft` both return empty. The BaDaaS overview README claims "A Lean port is in progress in cslib", but no supporting artifact was found (Section 15).

## 4. Property classes covered
| Class | verified-consensus | iris-lean | CSLib FLP |
|---|---|---|---|
| Functional correctness | no | infrastructure (HeapLang WP) | no |
| Safety invariant | yes (`finalizedSafe`, `nested`, `finalizedMonotone`, `honestNeverSlashed`) | infrastructure | no |
| Liveness | yes (`confirmedLive`, `stableLive`, `finalized`, `available`) under recurrence and participation premises | no | impossibility (non-termination of some fair run) |
| Refinement | no | no | no |
| Other | accountable safety (`certificatesAccountable`, `finalizedAccountable`), asynchrony resilience | adequacy of the program logic | impossibility theorem |

## 5. Link to implementation code
- verified-consensus: none. The Lean model is the executable artifact (`lean-is-impl` only in the sense that the model is executable). No Ethereum client code is connected. The link to the paper is manual: `docs/MODEL_MAP.md` maps each Section 7 item, and `docs/MODELING_CHOICES.md` lists "representation" and "idealization" differences.
- iris-lean: HeapLang is a toy language. No link to production code.
- CSLib FLP: not applicable (impossibility result).

## 6. Trusted computing base and soundness hygiene
Static counts with comments and strings stripped:
| Repo | files / lines | sorry | admit | axiom | native_decide | opaque | partial def |
|---|---|---|---|---|---|---|---|
| verified-consensus | 1,039 / 333,281 (Model 29/5,727; Statements 12/1,170; Proofs 851/309,574; Internal 139/10,243; Witnesses 8/6,567) | 0 | 0 | 0 | 5 | 0 | 1 |
| iris-lean | Iris 317 / 99,849; IrisMath 5 / 241 | 0 | 0 | 5 | 0 | 3 | 46 |
| CSLib FLP | 8 Lean files / 1,503 | 0 | 0 | 0 | 0 | 0 | 0 |
- verified-consensus: all 5 `native_decide` are in `DecoupledConsensusWitnesses/Outage.lean` (lines 1427, 1476, 1788, 1856, 1858). README: "The outage witness decides five finite checks with `native_decide`, so it also depends on `Lean.ofReduceBool`. The review theorem does not." `scripts/ReviewAxioms.lean` runs `#print axioms` on `concreteConsensus` and 22 supporting results. `scripts/verify.sh` checks that the axiom set is {`propext`, `Classical.choice`, `Quot.sound`}. The trusted spec surface is the two definition-only libraries, about 6,900 lines.
- iris-lean: the 5 `axiom`s are all in `Iris/IrisTest/Benchmarking/BenchmarkHeapLangList.lean` (benchmark stubs such as `isList_nil`, `nil_spec`), not in the library. A raw `sorry` at `Iris/Iris/BI/Sbi.lean:658` is inside a `/- -/` block.

## 7. Scale, effort, automation
- verified-consensus: about 333k lines of Lean, of which 309k are proofs, over about 5 months (repo created 2026-05-13; last commit 2026-10-12). README: "A full build takes about one hour on a recent laptop." CI (`verify.yml`) run 38307211827 on `main` at the cited commit completed with `success` in 5m43s (observed with `gh run list`; likely cached). AI involvement is visible in commit trailers and the AI-reviewer brief. Person-effort is not stated.
- iris-lean: about 100k lines. Effort not stated.
- CSLib FLP: 1,503 lines. Effort not stated.

## 8. Limitations stated by the authors (verbatim)
verified-consensus README "Scope and limitations":
> "**Generic statements.** The bundle is stated over an abstract interface. Only `Instantiation.lean` ties it to this protocol, so read that file with care: a wrong binding leaves the theorem true and empty."
> "**Stronger participation premises.** The premises hold at every time, not only at round samples. Thus finality needs `K ≥ 5`, not the paper's `K ≥ 4` (§7)."
> "**One outage.** The outage claim covers one asynchronous period, with bounded delivery before it and after GST. A run with more than one outage is not modelled (§5.1)."
> "**Witnesses show satisfiability.** The witness runs have one honest node and one silent Byzantine validator. They do not exercise equivocation or adversarial traffic."
> "**Outside the bundle.** Leak fairness, validator-client vote safety and height progress are proved in `ReviewTheorem.lean`, but they are not fields of `Consensus` (§10)."

## 9. Independent evidence
- verified-consensus: none external found. The repo invites AI audit (`docs/AI_AUDIT.md`), but no audit report is in the repo. CI success on `main` was observed.
- iris-lean, CSLib FLP: none specific found.

## 10. Reproducibility
Not built (orchestrator policy: shared sandbox disk was full, no toolchain or Mathlib downloads). For verified-consensus, GitHub Actions `verify` on `main` @ `d379754` reported `success`.

## 11. Generalisation
verified-consensus pattern: freeze a small definition-only "review surface" (model plus statements, about 6.9k lines) and bundle all claims into one structure-typed theorem. Mechanical scripts check the boundary, reachability of the statements, the bundle's shape and its axioms. Non-vacuity witnesses (concrete runs satisfying each regime) guard against empty premises, and there is a written brief for AI auditors. This applies to any protocol where the reviewer can be told to read only the surface. Requirement: the instantiation file must be read by a human or auditor, since "a wrong binding leaves the theorem true and empty". Specific to this case: no implementation link, and an unpublished paper as the reference.

iris-lean is infrastructure only. CSLib provides LTS/ω-execution foundations (used by lean-tla's `Bft/CslibBridge.lean`, REF-L18) and FLP, but no Raft.

## 12. Bearing on sub-claims (facts only, no verdict)
| Sub-claim | Fact from this reference | Source ID |
|---|---|---|
| H1 | A full PoS-style consensus protocol model with partial synchrony, Byzantine delivery, and safety/liveness/accountability claims is expressed in Lean 4 + Mathlib | S1 |
| H2 | 0 sorry and 0 axiom. The review theorem's axioms are checked by script to be the three standard ones. 5 `native_decide` are confined to a witness file | S1 |
| H3 | No link to a client implementation. iris-lean provides separation-logic infrastructure but has no distributed or network layer | S1, S2 |
| H4 | AI assistance is visible (Claude commit trailers). The repo ships an AI-reviewer audit brief | S1 |
| H5 | none | none |
| D | "machine-checked proof of its consensus guarantees", scoped to the Lean model of the paper's Section 7 | S1 |
| Raft in CSLib | No Raft port found in CSLib code, PRs or issues. CSLib does contain an FLP impossibility proof | S3 |

## 13. Conflicts
- verified-consensus paper version: README "Paper: `consensus.tex` at commit `9f5ed717ffac`" vs `docs/PROTOCOL.md` "Source: `consensus.tex` at commit `2f4b9d1c5e1a`".
- CSLib Raft: the BaDaaS `replication-algorithm-overview` README says "A Lean port is in progress in cslib". No Raft content appears in CSLib code, PRs or issues as of `76b7943` / 2026-11-07.

## 14. Sources
| ID | Title | URL | P/S | Depth | Summary |
|---|---|---|---|---|---|
| S1 | fradamt/verified-consensus @ d379754 (README, docs/AI_AUDIT.md, docs/MODELING_CHOICES.md, docs/PROTOCOL.md, Lean sources, scripts, CI) | https://github.com/fradamt/verified-consensus | P | CODE | Executable Lean model of a decoupled PoS consensus protocol with a 13-claim review theorem. 0 sorry, 0 axiom, axiom set checked by script. Premise ledger and limitations stated |
| S2 | leanprover-community/iris-lean @ 8293092 | https://github.com/leanprover-community/iris-lean | P | CODE | Lean 4 port of Iris (MoSeL, IProp, HeapLang, WP/adequacy). No distributed-systems layer |
| S3 | leanprover/cslib @ 76b7943, `Cslib/Computability/Distributed/FLP/` | https://github.com/leanprover/cslib | P | CODE | FLP impossibility (Völzer proof), 1,503 lines, 0 sorry. No Raft content, and no Raft PRs or issues |
| S4 | BaDaaS replication-algorithm-overview, 0033-raft README (search snippet) | https://github.com/BaDaaS/replication-algorithm-overview/blob/main/0033-raft/README.md | S | SNIPPET | Claims "A Lean port is in progress in cslib". Unsupported by S3 |

Access date for all: 2026-11-07.

## 15. Unverified leads (excluded from evidence)
- `solanaqubits/lean-verification-components` (Reservoir) says v0.4.3 "adds the Raft network induction and complete bridge, with eleven new Lean files". Not inspected.
- `nocaptheorem/RaftFormalModel`: "Lean 4 executable model of the Raft consensus algorithm"; its README says "the mathematical proofs (the `sorry` blocks) are left unimplemented for now". Not inspected.
- arXiv 2606.02019 "Federated Formal Verification ..." describes a TLAPS Raft proof tree with Lean 4 as one of six cross-check backends. Not read.
- Whether "decoupled consensus" corresponds to a named Ethereum proposal could not be confirmed, because the paper is not public.
