# REF-L21: Microsoft CCF Lean consensus specs and C++ trace replay

Kind: lean. Access date for all sources: 2026-11-07.

## 1. Identity
| Field | Value |
|---|---|
| Subject (protocol or algorithm) | CCF consensus protocol (CFT, Raft-derived, `src/consensus/aft/cft.h`), plus client-observable consistency, the KV store, and disaster recovery (DR) [S1]. |
| Spec it targets | No external standard. Lean executable models written to match the CCF C++ implementation; they replace the earlier TLA+ specs [S1, S3]. |
| Organisation / authors | Microsoft (CCF team). PR authors seen: `achamayou` (#8277, #8663) [S2, S3]. 36 of 40 commits touching `lean/` carry "Co-authored-by: Copilot" [S4]. |
| Primary URLs | https://github.com/microsoft/CCF/tree/main/lean ; https://github.com/microsoft/CCF/pull/8277 ; https://github.com/microsoft/CCF/pull/8663 |
| Repo, commit SHA, last commit date, licence | `microsoft/CCF` `6404230c2b1e0f1770f47ce95a4e3cb3a274c657`, 2026-11-06 ("Reduce memory used at the time of snapshot generation (#8689)"), Apache-2.0 [S1]. |
| Lean toolchain | `leanprover/lean4:v4.35.0`, Mathlib `v4.35.0`, `leanfmt` v0.4.5 (`lean/lean-toolchain`, `lean/consensus/lakefile.toml`) [S1]. #8277 used Lean and Mathlib 4.33.1; upgraded in #8570 [S2, S4]. |
| Maturity | Production CI gate in a production project: the `Lean` workflow is required on PRs touching `lean/`, `src/consensus/aft/**`, `src/kv/**` and related paths [S1 `.github/workflows/lean.yml`]. Work is about two months old (first merge 2026-09-09) [S2]. |

## 2. Claim as stated by the authors (verbatim)
- "We use agents to generate models and proofs that those models are correct. We use trace validation to align the models with the implementation. Formally, if B(X) is the set of behaviours permitted by X, then B(implementation) \subseteq B(Model) \subseteq B(Properties)." [S1 `lean/AGENT.md:3-5`]
- "To be clear being able to save 1 line of human readable code at the cost of 1000 machine generated and verified lines is a win." [S1 `lean/AGENT.md:8`]
- "They also provide stronger guarantees: their safety properties are machine-checked proofs that hold for all model instances, rather than results of bounded model checking or simulation with TLC." [S3, PR #8663]
- "A pass establishes compatibility of that finite trace under this replay policy, not universal C++ refinement, complete log-byte/KV equality, send completeness, fairness or liveness." [S5, PR #8584]
- "Machine checking establishes the stated mathematical claims, not that the model or assumptions correctly describe the implementation and deployment. Those remain human-review obligations." [S2, PR #8277]

Use of words (D): AGENT.md states an inclusion B(implementation) ⊆ B(Model) as the aim. The first inclusion is not proved anywhere in the tree. It rests on trace replay of finite test runs, which PR #8584 says does not establish "universal C++ refinement". "Verified" in AGENT.md refers to kernel-checked Lean proofs about the model.

## 3. What is actually formalised
Packages under `lean/` at HEAD (lines of `.lean`) [S1]:
| Package | Lines | Content |
|---|---|---|
| common | 661 | Transition systems, finite-or-infinite `Trace` with validity proof, network composition, `CCF.Parsec` event parsing |
| consensus | 11,685 | CFT node model (`Model/Node.lean`: `Node.step`, `receive`, `act`), 4 safety theorems, replay |
| consistency | 5,422 | Client history and ledger-branch model, 17 safety theorems, replay |
| disaster-recovery | 8,967 | DR local and global model, 7 theorems in `Properties.lean`, replay of SNP e2e logs |
| kv | 6,478 | KV store model, 37 theorems in `Properties.lean`, replay |
| Total | 33,213 | Of which files under `Proofs/` 23,790; replay and trace-validation files 2,593 |

Main consensus safety theorems, verbatim from `lean/consensus/Consensus/Properties.lean` [S1]:
- Line 26: `theorem committed_logs_prefix : forall (config : Finset NodeID), forall (nodes : List NodeID) (trace : GlobalTrace config nodes), forall (stateA stateB : Model.State), forall (left right : NodeID) (leftState rightState : NodeState), (stateA ∈ trace /\ stateB ∈ trace /\ (left, leftState) ∈ stateA.nodes /\ (right, rightState) ∈ stateB.nodes) -> leftState.committedLog <+: rightState.committedLog \/ rightState.committedLog <+: leftState.committedLog`
- Line 57: `theorem single_node_committed_log_monotonic : forall (config : Finset NodeID) (nodes : List NodeID), forall (pre post : Model.State), forall (nodeId : NodeID) (nodePre nodePost : NodeState), forall (action : Model.Action), ((pre.nodes.map Prod.fst).Nodup /\ (Model.transitionSystem config nodes).step pre action = some post /\ (nodeId, nodePre) ∈ pre.nodes /\ (nodeId, nodePost) ∈ post.nodes) -> nodePre.committedLog <+: nodePost.committedLog`
- Line 85: `theorem election_safety : forall (config : Finset NodeID), forall (nodes : List NodeID) (trace : GlobalTrace config nodes) (state : Model.State), forall (left right : NodeID) (leftState rightState : NodeState), (state ∈ trace /\ (left, leftState) ∈ state.nodes /\ (right, rightState) ∈ state.nodes /\ leftState.role = .primary /\ rightState.role = .primary /\ leftState.currentTerm = rightState.currentTerm) -> left = right`
- Line 114: `theorem committed_frontier_is_signature : forall (config : Finset NodeID), forall (nodes : List NodeID) (trace : GlobalTrace config nodes) (state : Model.State), forall (node : NodeID) (nodeState : NodeState), (state ∈ trace /\ (node, nodeState) ∈ state.nodes /\ 0 < nodeState.commitIndex) -> isSignatureAt nodeState.log nodeState.commitIndex = true`
- Each has an adjacent `_witness` lemma (lines 41, 71, 100, 124) giving a non-vacuity execution. The `election_safety_witness` shows two primaries in different terms; PR #8583 notes this is "not a violation of election safety" [S1, S6].
- `GlobalTrace config nodes` is `CCF.Execution.Trace (Model.transitionSystem config nodes)`, a finite or infinite sequence of actions with a validity proof (`common/CCF/Execution.lean:15-48`) [S1].

Other packages, examples [S1]: `consistency/consistency/Properties.lean:29 committed_rw_ordered_serializable`, `:51 committed_rw_ordered_real_time` (described as "the primary consistency guarantee" in `consistency/README.md`); `disaster-recovery/DisasterRecovery/Properties.lean:19 quorum_opener_unique`, `:36 quorum_open_preserves_commit`; `kv/Kv/Properties.lean:154 transaction_snapshot_serial`, `:597 durable_cut_survives_rollback`.

Scope exclusions (verbatim, `lean/consensus/README.md`): "Node join and snapshot installation are not modeled." "Actions are local inputs (including timeouts) or delivery of a queued message; no clock is modeled." [S1] Consistency: "There is no refinement proof connecting it to the [CCF consensus protocol model]" [S1 `lean/consistency/README.md`].

## 4. Property classes covered
| Class | Status | Evidence |
|---|---|---|
| Functional correctness | partial | KV read/write lemmas (`read_your_write`, `apply_atomic`) on the KV model [S1]. |
| Safety invariant | yes (model) | Consensus 4, consistency 17, DR 7, KV 37 theorems; all proved for all model instances [S1]. |
| Liveness | no | "There are no liveness properties or fairness assumptions." (consensus README); "Liveness properties are out of scope." (DR README); "The model covers safety only, not liveness" (consistency README) [S1]. DR `quorum_advance_opens` and `aligned_opening_timeout_completes` are single-step facts, not progress properties [S1]. |
| Refinement | no (tested only) | No proof linking C++ to model; replay of finite traces [S1, S5]. No proof linking the consistency model to the consensus model [S1]. |
| Symbolic security | no | DR `Validation.accepted/rejected` "do not prove quote, certificate, or other cryptographic checks" [S2]. |
| Computational security | no | Not addressed. |
| Parsing | no | Replay parsers are trusted code (`CCF.Parsec`, `partial def many`) [S1]. |
| Constant-time | no | Not addressed. |

## 5. Link to implementation code
Mechanism: trace validation by replay in Lean (index category: trace-validation).
- Emission in C++: `src/consensus/aft/cft.h` calls `trace::` events at 24 points (e.g. line 838 `trace::tick`, 2112 `trace::become_leader`, 2472 `trace::commit`), declared in `cft_trace_msgpack.h` with tag `ccf.cft` and serialised to msgpack via `ccf::tracing::emit` into a Fluentd sink [S1]. These calls are in the production consensus header.
- Driver: `cft_driver` (`src/consensus/aft/test/driver.cpp`, `driver.h`) runs the real `cft.h` with stub ledger and network (`logging_stub.h`). It also emits each scenario command as a `cmd` event (driver.cpp:86) [S1].
- Scenarios: 56 files in `tests/cft_scenarios/`, run by `tests/cft_scenarios_runner.py`, which writes `consensus/raw/<scenario>.ndjson` [S1]. PR #8584: "replayed all 56 scenarios successfully" [S5].
- CI (`.github/workflows/lean.yml`) [S1]:
  1. Job `cft_traces`: builds `cft_driver` (Debug, `-DVERBOSE_LOGGING=ON`), runs `ctest -R "^cft_scenario_test$"`, uploads `*.ndjson`.
  2. Job `cft_replay` (needs `cft_traces`): `lake build --wfail consensus-replay`, then for each trace runs `lean/consensus/.lake/build/bin/consensus-replay "$trace"`.
  3. Matrix job `packages` calls `lean-package.yml`: `lake check --inadvisably-no-sandbox` (kernel and axiom check), `lake exe check-imports`, format, `lake lint`, and positive and negative replay fixtures (consensus: 1 valid, 7 invalid fixtures that must exit 1).
  4. Analogous `kv_traces`/`kv_replay` (instrumented `kv_test`, 8 seeds) and `consistency_traces`/`consistency_replay` (e2e `consistency_trace_validation`).
  5. Job `models` requires every one of these jobs to succeed.
- What replay checks (verbatim, `lean/consensus/replay/README.md`): "Every event must match a named rule ... Actions must be enabled by the model. After bootstrap, each rule emits one read-only observation before its model actions, comparing the native group's entry snapshot with the current model state." "Delivery is strict FIFO per directed node pair, preserving multiplicity and delivering the queued **model payload**, not reconstructing unrecorded native bytes." Consensus README: it "never repairs model state." [S1]
- Guarantee: for each finite captured trace, there exists a model execution (under FIFO per-pair delivery) whose states agree with the recorded fields. PR #8584: "A FIFO failure does not rule out a compatible execution under another delivery order." [S5]. DR replayer README: "It does not show behaviour that no record shows, or liveness." [S1]

## 6. Trusted computing base and soundness hygiene
| Item | Count / status | Source |
|---|---|---|
| `sorry` | 0 (grep of `lean/**/*.lean`) | S1 |
| `axiom` declarations | 0 | S1 |
| `native_decide` | 0 | S1 |
| Axiom allowlist | `propext`, `Classical.choice`, `Quot.sound` enforced by `lake check` in CI | S1 `replay/README.md` |
| `partial def` | 4, all in trace parsing or replay drivers (`common/CCF/Parsec.lean:92,100`, `kv/Kv/TraceValidation/Records.lean:77`, `disaster-recovery/replayer/Main.lean:44`) | S1 |
| Build flags | `-DwarningAsError=true` (so `sorry` warnings fail), `-Dtrace.Kernel=true`, `autoImplicit = false` | S1 `lakefile.toml` |
| Sandbox | "It runs without sandboxing here, so build-time code must be trusted." | S1 consensus README |
| Unverified translator | Model is hand- or agent-written from C++, not extracted. C++ trace emission, msgpack serialiser, Fluentd sink, Python runner, and Lean replay parser and reduction rules are trusted for the replay result. | S1 |

## 7. Scale, effort, automation
- 33,213 Lean lines; about 72% under `Proofs/` [S1]. 267 top-level `theorem`/`lemma` declarations matched by a line-start grep (indented declarations not counted) [S1].
- #8277 (DR model): +3,901 lines, "9 explicitly stated system properties and 124 supporting lemmas"; "1,079 declarations under `DisasterRecovery` within the axiom allowlist" [S2]. #8583 (consensus proofs): +8,261 lines [S6]. #8584 (replay): +1,605 lines [S5].
- Timeline (merge dates) [S4]: #8277 DR model 2026-09-09; #8282 C++ DR trace validation 2026-10-09; #8578 consensus model 2026-10-20; #8583 consensus proofs 2026-10-22; #8584 consensus replay 2026-10-29; #8633 consistency 2026-10-29; #8339 KV 2026-11-03; #8664 consistency trace validation 2026-11-04; #8663 TLA+ removal 2026-11-05. Several PRs on 2026-11-05/06 report 2 to 5 times faster proof checking ("proofs only").
- Automation: AGENT.md says agents generate models and proofs [S1]. 36 of 40 commits under `lean/` are co-authored by Copilot [S4]. #8277 marks `Proofs/` files `linguist-generated`, stating "Their proof steps do not normally need line-by-line human review once the checks pass" [S2].
- Human review boundary: "Review the model, full statements and proof links in `Properties.lean`, and the property helpers used by those statements." [S1 AGENT.md]

## 8. Limitations stated by the authors (verbatim)
- "A pass establishes compatibility of that finite trace under this replay policy, not universal C++ refinement, complete log-byte/KV equality, send completeness, fairness or liveness." [S5]
- "Send observations check the latest pending packet but need not enumerate every emission exactly once. Finite traces may end without a final observation or with packets still pending." [S5]
- "Cache observations compare exactly the native first/last markers, not unrecorded interior entries." [S5]
- "These are safety proofs of the CCF-Raft Lean model, not proofs about generic Raft, C++ or liveness." [S6]
- "Committed-TxID preservation requires explicit `DurableCommit` and `FullGossipSelection` premises. A quorum opening alone does not establish full-gossip selection." [S2]
- "These are regression examples, not exhaustive checking." (canonical action traces) [S1 consensus README]
- #8683: "If the model silently forbids a real CCF behaviour, every theorem still holds, just vacuously for that behaviour." Mutation testing found model changes "that passed every proof and witness before this PR" [S7].

## 9. Independent evidence
- None external found. All sources are CCF repository artefacts and PRs.
- Internal cross-check: #8279 validated the Lean DR model against the earlier Rust Stateright model before that model was removed (#8280) [S4].
- Internal check of vacuity: #8683 ran `jubnzv/Mutate.lean` over 297 mutation sites of the consistency model and added witnesses for the gaps [S7].
- Earlier TLA+ trace validation in CCF is covered in REF-C08 (paper reports bugs found). No public report yet of bugs found by the Lean replay; #8277 mentions "the accepted-unexpected-location behavior exposed in #8241", kept intentionally in the model [S2].

## 10. Reproducibility
Not run: no build attempted (instruction: static inspection only; Mathlib build too large for the sandbox). Commands from the repo: `cd lean/consensus && lake exe cache get && lake build --wfail --fail-fast && lake exe check-imports && lake lint`; native traces: `cmake -S ../.. -B /tmp/cft-build -GNinja && cmake --build /tmp/cft-build --target cft_driver && ctest --test-dir /tmp/cft-build -VV -R '^cft_scenario_test$'`, then `./.lake/build/bin/consensus-replay <trace>.ndjson`. Static inspection used a shallow clone (`--depth 1 --filter=blob:limit=1m`), rg, wc, and `gh pr view`; the clone was deleted.

## 11. Generalisation
Pattern: an executable Lean transition-system model of the C++ protocol; safety theorems over all finite and infinite traces of the model; implementation conformance from instrumented C++ traces replayed through the model with read-only state observations; CI gating on both.
Preconditions:
- Trace emission hooks at the implementation's protocol steps, with enough state (here node id, view, last_idx, commit_idx, leadership and membership state, retirement fields, and only first/last committable indices) [S1 `cft_trace_msgpack.h`].
- A deterministic driver that runs the real implementation with stubbed I/O [S1].
- An existing set of scenario tests (56 here) to produce traces.
- Model shape matched to implementation steps, so each event maps to a named rule.

## 12. Bearing on sub-claims (facts only, no verdict)
| Sub-claim | Fact from this reference | Source ID |
|---|---|---|
| H1 | CCF consensus, consistency, KV and DR protocols were formalised as executable Lean models, 33,213 lines total. | S1 |
| H2 | 65 public safety theorems proved over all model traces; 0 sorry, 0 axioms, 0 native_decide; axiom allowlist enforced in CI. No liveness. | S1 |
| H3 | C++ link is replay of finite driver and e2e traces in CI; authors state it is not universal C++ refinement. | S1, S5 |
| H4 | Developed in about two months (2026-09-09 to 2026-11-05); 36 of 40 `lean/` commits co-authored by Copilot; AGENT.md says agents generate models and proofs. | S1, S2, S4 |
| H5 | A production team replaced TLA+/TLC with Lean, giving "stronger guarantees" and "wider scope" as reasons. | S3 |
| D | AGENT.md writes B(implementation) ⊆ B(Model) as the formal aim; replay docs disclaim refinement. | S1, S5 |

## 13. Conflicts
- AGENT.md states "B(implementation) \subseteq B(Model)" [S1], while PR #8584 states a pass is "not universal C++ refinement" [S5]. The first is an aim, the second a limit; the tree contains no proof of the inclusion.
- PR #8663 says it "Removes TLA-only trace tooling: the `consistency_trace_validation` e2e test, `tests/tvc.py`, the `samples/apps/basic_tv` app" [S3]. At HEAD `tests/consistency_trace_validation.py`, `tests/tvc.py`, `samples/apps/basic_tv/` and the CMake test `consistency_trace_validation` exist and feed the Lean `consistency_traces` job [S1]. #8664 (Lean consistency trace validation, merged 2026-11-04) likely kept or re-added them; not resolved.
- PR #8663 refers to `lean/ccf-raft`; at HEAD the package is `lean/consensus` (renamed in #8657) [S3, S4].
- `lean/consensus/README.md` says replay's "assurance limits are documented there" (in `replay/README.md`), but that file at HEAD has no section on assurance limits; the detailed limits are in the PR #8584 description [S1, S5].
- C++ trace sink can drop events (`FluentdSink::record_drop`, `dropped` counter in `src/tracing/fluentd_sink.h`) [S1]. Whether a dropped event can make a replay pass or only fail was not established.

## 14. Sources
| ID | Title | URL | P/S | Depth | Summary |
|---|---|---|---|---|---|
| S1 | microsoft/CCF repository at 6404230 (`lean/`, `.github/workflows/lean.yml`, `lean-package.yml`, `src/consensus/aft/`, `src/tracing/`) | https://github.com/microsoft/CCF/tree/6404230c2b1e0f1770f47ce95a4e3cb3a274c657 | P | CODE | Lean models, theorems, replay, CI, C++ trace emission. |
| S2 | PR #8277 "Add the C++-aligned Lean disaster recovery model" | https://github.com/microsoft/CCF/pull/8277 | P | FULL | First Lean merge; review boundary, proof limits, axiom audit. |
| S3 | PR #8663 "Remove TLA+ specifications and associated tooling" | https://github.com/microsoft/CCF/pull/8663 | P | FULL | Motivation and scope of TLA+ removal (75 files, -7,040 lines). |
| S4 | CCF merged PR list and `lean/` commit history (GitHub API) | https://github.com/microsoft/CCF/commits/main/lean | P | FULL | Complete API listing of the 40 commits touching `lean/` (titles, dates, Co-authored-by trailers); metadata only. |
| S5 | PR #8584 "Lean-only CCF Raft trace reduction and replay" | https://github.com/microsoft/CCF/pull/8584 | P | FULL | Replay design and "What replay establishes". |
| S6 | PR #8583 "Prove CCF-Raft safety properties in Lean" | https://github.com/microsoft/CCF/pull/8583 | P | FULL | Four consensus theorems, witness design, scope disclaimer. |
| S7 | PR #8683 "Lean consistency: witnesses for gaps found by mutation testing" | https://github.com/microsoft/CCF/pull/8683 | P | FULL | Mutation testing (297 sites) exposed 8 model gaps that passed every proof and witness; survivors in `Transitions.lean` cut from 15 to 6; merged 2026-11-06. |

## 15. Unverified leads (excluded from evidence)
- PR #8241 (superseded DR work exposing "accepted-unexpected-location behavior") not read.
- PR #8282 (C++ DR trace validation with Lean), #8339 (KV), #8664 (consistency trace validation) not read in full.
- Issue #6183 (closed by #8663) not read.
- Whether the Lean replay has found any real C++ bug: not found in the sources read.
- Whether the human-reviewed share of the Lean code is measured anywhere: not found.
