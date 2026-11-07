# REF-C10 Aneris / Trillium / Grove / Fairneris (Rocq + Iris comparator)

Comparator dossier. Every result below is a Rocq (Coq) result about Rocq/Iris developments. Nothing here is a Lean result. The existence of iris-lean (see REF-L20) or of Perennial's Lean branch (see REF-L22) does not transfer any of these theorems to Lean. §11 records what exists on the Lean side.

## 1. Identity
| Field | Value |
|---|---|
| Name | Aneris (distributed separation logic); Trillium (intensional-refinement framework, with its Aneris and Fairis instantiations); Grove (Perennial extension for distributed Go); Fairneris (liveness-preserving refinement for distributed systems) |
| Organisation | Aarhus University (Logsem; Birkedal, Timany et al.), with MPI-SWS and Cambridge co-authors; MIT PDOS (Grove: Sharma, Jung, Tassarotti, Kaashoek, Zeldovich) |
| Repositories | `logsem/aneris` @ `59ae37946838df77fb2c573803b68462a3af9d0f` (2025-10-31; pins `external/trillium` at `43d0cf4`); `logsem/trillium` @ `43d0cf4021cc1d08da008c5aa104aa6f5ea75f17` (2024-01-16); `mit-pdos/perennial` tag `sosp2023` @ `820a2a9ab088d298f9854961c55c010d6ea1858d` (2023-09-10) |
| Proof assistant | Rocq/Coq + Iris (`coq-aneris.opam`: `"coq" { (= "8.16.1") \| (= "dev") }`) |
| Licence | MIT (aneris, trillium, perennial `LICENSE` files) |
| Publications | Aneris, ESOP 2020 (S4); Trillium, POPL 2024 (S1); Grove, SOSP 2023, extended version (S2); Fairneris, HAL preprint hal-05647709, submitted 7 Jun 2026 (S3) |
| Maturity | research |
| AI provenance | `none-found`. No `CLAUDE.md`, `AGENTS.md` or similar agent config in the three inspected checkouts. Shallow clones carry no commit-trailer history; trailers were not checked |

## 2. Claim as stated by the authors (verbatim)
- Aneris (S4, abstract): "we present Aneris, a novel framework based on separation logic supporting modular, node-local reasoning about concurrent and distributed systems. The logic is higher-order, concurrent, with higher-order store and network sockets, and is fully mechanized in the Coq proof assistant."
- Trillium (S1, abstract): "we introduce Trillium, a language-agnostic separation logic framework for showing intensional refinement relations between traces of a program and a model. ... We also instantiate Trillium with a distributed language and obtain an extension of Aneris, a distributed separation logic, which we use to show refinement relations between distributed systems and TLA+ models."
- Grove (S2, abstract): "Grove is the first to handle time-based leases, including their interaction with reconfiguration, crash recovery, thread-level concurrency, and unreliable networks. This paper uses Grove to verify several distributed system components written in Go, including vKV, a realistic distributed multi-threaded key-value store."
- Fairneris (S3, title): "A Higher-Order Separation Logic for Verifying Liveness-Preserving Refinements of Distributed Systems".

## 3. What is actually formalised
Each refinement relation and each assumption is listed separately (Phase 1c instruction).

### 3.1 Refinement relations
R1. Trillium generic `continued_simulation` (`trillium/program_logic/traces.v:119-149` @ `43d0cf4`). This is a coinductive relation between a program execution trace and an auxiliary model trace, parameterised by a user relation `φ`. Every program extension must be matched by some model extension that preserves `φ`. It is a safety-style (prefix) simulation and does not by itself preserve liveness.

R2. Aneris → Paxos TLA+ model (`aneris/examples/consensus/paxos_adequacy.v` @ `59ae379`):
- `paxos_simulation` (line 290) proves `continued_simulation simulates` for the runner program.
- `simulates` (lines 96-105) requires two things. Model steps must be stutter-or-`PNext`. The model's message set must agree with the messages actually sent by the parties (`messages_model_agree δ.(msgs) (messages_sent_from_parties smh)`).
- The model is a Coq STS that S1 says was "manually translated" from TLA+ (§6).

R3. Aneris → two-phase-commit TLA+ model (`aneris/examples/transaction_commit/two_phase_adequacy.v`): `tpc_simulation` (lines 111-114), with its own `simulates` (line 89) relating executions to `TPNext` steps.

R4. Grove has no model-refinement theorem of the R2/R3 kind. Its top-level statement is a distributed adequacy theorem: proved node-local WPs imply that the distributed Go system does not get stuck and the global invariant `φinv` holds (`src/goose_lang/ffi/grove_ffi_adequacy.v:13` `grove_ffi_dist_adequacy`, `:42` `..._failstop`, @ `820a2a9`). Client-facing specs (for example linearizable KV operations) are separation-logic specs proved per component.

R5. Fairis fair-termination preservation (`fairis/fuel_termination.v:11-24` @ `43d0cf4`): `continued_simulation_fair_termination` assumes the model is fairly terminating and the program is related by `continued_simulation` with a fuel-decreasing relation. It concludes that every fair program execution terminates. Fairis is an instantiation for a concurrent (shared-memory) language, not for the distributed language.

R6. Fairneris (S3): a liveness-preserving refinement between distributed programs and models, under a network-fairness assumption (A-F2 below). I found no public code at a version matching the 2026 preprint (§10, §15). R6 is author-reported only.

### 3.2 Network assumptions (each stated separately)
- A-N1 Aneris (current code). Messages may be dropped and duplicated. `aneris/aneris_lang/lang.v:1032` `MessageDuplicateStep` adds a copy of an in-flight message (`state_ms σ ⊎ {[+ m +]}`). `:1040` `MessageDropStep` removes one. `:1020` `MessageDeliverStep` delivers any message addressed to a socket. These are UDP-style semantics with no ordering.
- A-N2 Aneris (ESOP 2020 paper). S4: "we assume duplicate protection" (see §13 Conflicts).
- A-N3 Grove. In the sosp2023 code, `SendOp` (`src/goose_lang/ffi/grove_ffi.v:172-183`) adds the message to the destination's set (`ms ∪ {[Message c_l data]}`) and may non-deterministically report an error before or after (`err_early`, `err_late`). `RecvOp` (`:185-194`) returns `None` (error) or any message in the set from the right sender, without removing it. As a result, messages can be lost from the receiver's perspective (receive may always error), received more than once, and reordered. S2 describes its network as "unreliable".
- A-N4 Fairneris. S3: "it is possible for a message to never be delivered, preventing liveness" without fairness. Liveness is proved under A-F2.

### 3.3 Fairness assumptions
- A-F1 Fairis: fair scheduling of threads, which is premise "fair execution" of R5. Shared memory only.
- A-F2 Fairneris: "a flexible network fairness assumption". S3 glosses a fair network as one where "it does not drop all messages", and says it defines "new network fairness conditions at both the operational semantics level and the model level" (S3 §1). This is author-reported; I could not inspect a matching definition in public code.
- Trillium's Aneris instantiation (R2, R3) and Grove (R4) prove no liveness, so they carry no fairness assumption. S2 states: "One limitation is that Grove is only able to verify safety".

### 3.4 Crash assumptions
- A-C1 Aneris/Trillium. A lexical search for `crash|restart` in `aneris/aneris_lang/lang.v` and `network.v` finds no node-crash step. Node crashes are not modelled in the inspected semantics.
- A-C2 Grove. Nodes may crash and restart, with durable files.
  - `grove_dist_adequate` (`src/program_proof/simplepb/apps/closed_proof.v:21-29`) takes `init_restart := fst ebσ` for each node. A crashed node restarts with its main program.
  - The paper (S2): "independent node crashes" are among the new reasoning principles. The trusted "Filesystem library" models durable storage.
  - A failstop variant (`grove_ffi_dist_adequacy_failstop`, `grove_ffi_adequacy.v:42`) requires a WP only for the initial threads.
- A-C3 Fairneris. S3 says of Grove's crash recovery and reconfiguration: "An interesting and important future work is to prove liveness properties of such distributed systems." The S3 sections I read give no crash model for Fairneris itself. S3's comparison table did not extract legibly (check and cross marks were lost), so it is not used.

### 3.5 Time and clock assumptions
- A-T1 Grove. `GetTimeRangeOp` (`grove_ffi.v:244-254`) returns a pair bounding the global time, so each node's clock has bounded uncertainty around a single global time. Lease reasoning depends on this. `GetTscOp` (`:235`) gives a monotone local counter.
- Aneris/Trillium/Fairneris have no clock primitive in the inspected material.

## 3a. Predicate semantics
Paxos (Aneris/Trillium). Model-level conclusion, `aneris/examples/consensus/paxos_model.v:745-746`, with `Chosen` at `:169-172`:
```coq
  Theorem paxos_correct s v1 v2 :
    rtc PNext PInit s → Chosen s v1 → Chosen s v2 → v1 = v2.
  ...
  Definition ChosenAt s bal val :=
    ∃ Q, QuorumA Q ∧ ∀ a, a ∈ Q → VotedFor s a bal val.

  Definition Chosen s val := ∃ bal, ChosenAt s bal val.
```
Implementation-level corollary, `paxos_adequacy.v:412-423`:
```coq
Definition ChosenImpl (s : state) (v : Value) :=
  ∃ b Q,
    QuorumA Q ∧
    ∀ a, a ∈ Q →
         ∃ (l : Learner),
           mkMessage (`a) (`l) (msg2b_ser b v) ∈ s.(state_ms).

Corollary paxos_correct_impl v1 v2 tp σ ex:
  trace_starts_in ex ([runner_expr], init_state) →
  trace_ends_in ex (tp, σ) →
  valid_exec ex →
  ChosenImpl σ v1 → ChosenImpl σ v2 → v1 = v2.
```

Strongest premises:
- The execution is of the fixed runner `runner_topo` (`paxos_runner.v`), which has a fixed set of proposers, acceptors and learners.
- Execution steps follow A-N1 (no crashes, A-C1).
- The Paxos model is the hand-translated STS.
- `classical_instances` imports Trillium's `FunExt`, `PropExt` and `Choice` axioms (`trillium/prelude/classical.v:3,6,18`).

Plain language: in the final state of every valid finite execution of this specific Aneris runner, no two different values both have a quorum of serialised 2b messages in the in-flight message multiset `state_ms`. Because `MessageDropStep` removes messages from `state_ms`, the predicate concerns messages currently in flight, not all messages ever sent. Not guaranteed:
- Liveness (a value is eventually chosen).
- Other topologies.
- Crash behaviour.
- Correspondence of the AnerisLang program to the OCaml source. `ml_sources/` are translated by the unverified `o2a` tool (README lines 66-80).

Two-phase commit. `tpc_consistent` (`two_phase_adequacy.v:186-196`) concludes `¬ (st1 = ABORTED ∧ st2 = COMMITTED)` for two resource managers whose state messages `m1`, `m2` are in `σ.(state_ms)` at the end of a valid execution of the runner. It is obtained through R3, and the same premises and exclusions apply.

Grove vKV/bank. `closed_bank` (`closed_proof.v:109-156`) instantiates `grove_dist_adequate` with `φinv := λ _, True`. The guarantee is therefore that no node ever gets stuck (Goose assertion failure or undefined behaviour is "stuck"). The bank auditor's balance-sum check `SimpleAudit` (`external/Goose/github_com/mit_pdos/gokv/bank.v:89-98` @ `820a2a9`) panics when the sum is wrong. So "never stuck" implies the auditor never observes a wrong total.

Premises:
- The Goose translation is faithful.
- A-N3, A-C2 and A-T1 hold.
- The trusted network (120 lines) and filesystem (50 lines) library models are faithful (S2 Table).

Not guaranteed: liveness or availability (S2 says safety only), and behaviour of the compiled Go binary beyond the Goose semantics.

Fairis R5. Conclusion: every fair execution of the program terminates. Premises: the model is fairly terminating, fuel decreases along the simulation, and thread scheduling is fair (A-F1).

## 4. Property classes covered
| Class | Status | Where |
|---|---|---|
| Safety (consensus agreement, atomic commit) | proved | R2 (Paxos), R3 (2PC), Aneris/Trillium @ `59ae379` |
| Safety (distributed KV, leases, reconfiguration) | proved | Grove R4 @ `820a2a9` |
| Liveness (concurrent, fair scheduling) | proved | Fairis R5 (shared-memory language only) |
| Liveness (distributed) | author-reported | Fairneris R6 (no matching public code inspected) |
| Refinement | proved | R2, R3 (trace simulation to TLA+-derived STS) |
| Functional | partial | Grove per-component specs (KV linearizability per S2) |
| Security (symbolic/computational) | no | n/a |

## 5. Link to implementation code
- Aneris: programs are AnerisLang (an ML-like language with sockets). OCaml sources in `ml_sources/` are converted by `ocaml2lang`/`o2a` (README). The tool is outside the TCB statement, and the proofs are about the AnerisLang terms. Mechanism: `manual` (source-to-model translation by an unverified tool).
- Grove: Go source translated to GooseLang by Goose. The proofs are about GooseLang. Mechanism: `extraction`-like source translation, recorded as `manual`. The schema has no "source-to-semantics translator" value, so this choice is my nearest fit.
- Model side: S1 says the TLA+ specs were "manually translated" into Coq STSs (`model-only` for the TLA+ link).

## 6. Trusted computing base and soundness hygiene
Static lexical counts (not a proof-closure audit):
- logsem/aneris @ `59ae379`: 476 `.v` files, about 114,012 lines (including the empty submodule directories). 2 `Admitted` lines, at `aneris/examples/dscm/implementations/one_server/proof/one_server_client_proxy_proof.v:150` and `aneris/examples/crdt/statelib/proof/time.v:16`. 14 `Axiom`/`Parameter`/`Conjecture` lines, all in `aneris/examples/echo_groups/proof.v`. None of these files is under `examples/consensus` or `examples/transaction_commit`. I did not run `Print Assumptions` to confirm that they are absent from the Paxos/2PC closure.
- logsem/trillium @ `43d0cf4`: 44 `.v` files, about 19,675 lines. 0 `Admitted`. 3 `Axiom` lines (`FunExt`, `PropExt`, `Choice` in `trillium/prelude/classical.v`).
- perennial `sosp2023` @ `820a2a9`, `src/program_proof/simplepb`: 36 files, about 21,038 lines. 0 `Admitted`/`admit` lines in `simplepb`, `grove_shared`, `grove_ffi.v` and `grove_ffi_adequacy.v`. `simplepb/apps/print_assumptions.v` runs `Print Assumptions closed_proof.closed.closed_bank`; I did not run it.

Trusted pieces stated by authors or code:
- Trillium/Aneris: the AnerisLang semantics, Coq + Iris, the classical axioms, the TLA+→STS hand translation (S1: "We have manually translated the TLA+ protocol specifications into STSs in Coq"), and `o2a` for OCaml-sourced programs.
- Grove (S2 table): "Network library 120 Trusted", "Filesystem library 50 Trusted". Goose and the GooseLang semantics are inherited from Perennial; the S2 sections I read do not list Goose as a separate trusted row.

## 7. Scale, effort, automation
- Grove (S2): "Grove's extensions to Perennial involved 1,597 lines of Coq proof". The per-component table lists, for example, vKV 233 code / 1,574 spec+proof, paxos 492 / 5,600 and replica 578 / 8,093.
- Aneris Paxos example: `aneris/examples/consensus/*.v` is 3,006 lines. 2PC: `transaction_commit/*.v` is 1,293 lines.
- All proofs are interactive Iris Proof Mode proofs, with no AI involvement reported.

## 8. Limitations stated by the authors (verbatim)
- Grove (S2): "One limitation is that Grove is only able to verify safety".
- Trillium (S1): "We have manually translated the TLA+ protocol specifications into STSs in Coq".
- Aneris ESOP 2020 (S4): "We assume duplicate protection to simplify our examples, as otherwise the code of all of our examples would have to be adapted to cope with duplication of messages."

## 9. Independent evidence
No independent rebuild by me (no toolchains policy). These are peer-reviewed venues (ESOP 2020, POPL 2024, SOSP 2023). Fairneris is an unrefereed preprint. Grove is used in the MIT 6.5840 reading list (S2 hosted on pdos.csail.mit.edu/6.824); this is not evidence of correctness.

## 10. Reproducibility
- Aneris: pinned opam/Coq 8.16.1 and submodules (iris, stdpp, trillium @ `43d0cf4`, actris, paco, record-update).
- Grove: tag `sosp2023`.
- Fairneris: the public `fairneris` (`aef7b2a`, 2023-01-25) and `fairneris-stenning` (`89a740d`, 2023-06-22) branches of logsem/aneris predate the 2026 preprint. I did not find code matching the preprint's case studies there. S3 cites an anonymised Rocq mechanisation.
- Not rebuilt (`evidence_level: static-inspected`).

## 11. Generalisation
- Scope axes:
  - quantifier: some (named example programs);
  - protocol class: crash-free consensus/commit (Aneris), crash-recovery primary/backup with leases (Grove);
  - property: safety; liveness only for Fairis (shared memory) and Fairneris (author-reported);
  - semantics: source-level languages (AnerisLang, GooseLang);
  - assumptions: as in §3.2-3.5;
  - envelope: expert Iris effort (thousands of lines per protocol).
- Lean status (facts only, not a transfer):
  - iris-lean has no distributed layer (REF-L20).
  - Perennial's default branch `lean` @ `84d5380` (2026-11-05) states: "There is no distributed program logic", while describing `goose_dist_adequacy` and crash adequacy in `GooseLang/DistAdequacy.lean`. That README was read for this point only. REF-L22 owns Perennial-Lean.
  - No Grove, Aneris, Trillium or Fairneris result is established in Lean by any source in this dossier.

## 12. Bearing on sub-claims (facts only, no verdict)
| Sub-claim | Fact from this reference | Source ID |
|---|---|---|
| H1 | (Rocq, not Lean) Separation logics express node-local concurrency, sockets with drop and duplication, crash-recovery, bounded-uncertainty clocks and TLA+-derived models | S1, S2, S5, S7 |
| H2 | (Rocq) Paxos and 2PC refinement-based safety, and a lease-based replicated KV's safety, are proved with 0 lexical `Admitted` on the inspected paths. Distributed liveness is only author-reported (Fairneris) | S1, S2, S3, S5, S7 |
| H3 | (Rocq) Proofs target source-level semantics: AnerisLang (OCaml via unverified `o2a`) and GooseLang (Go via Goose). Network and filesystem models are trusted | S2, S5, S7 |
| H5b | Not applicable: these are full proofs, not spec-only checking | none |
| D | "verify" means Iris WP plus adequacy to a stated trace/invariant property; "refinement" means `continued_simulation` to a model STS | S1, S6 |

## 13. Conflicts
- Trillium (S1) vs REF-C02 (IronFleet).
  - S1 (§6 Related Work): "IronFleet uses the Dafny verifier to verify implementations and encodes the refinement of an STS in preconditions and postconditions of programs but does not support node-local concurrency. IronFleet uses a pen-and-paper argument for proving liveness of simple programs (programs that consist of a simple event loop which calls terminating event handlers)."
  - REF-C02 quotes IronFleet as claiming "the first machine-verified liveness proofs" and records a mechanised IronRSL liveness lemma (`LivenessProof.i.dfy:192-198`, with one live `assume TimeNotZeno`). REF-C02 also records that IronFleet's reduction argument is informal ("we sketch an informal argument").
  - Possible reading, not settled here: S1's "pen-and-paper" may refer to the informal reduction step, not to the protocol-level liveness proof. On concurrency, REF-C02 records a reduction obligation that lets atomic host steps stand for interleaved I/O. This is a single event loop per host, which is consistent with "no node-local concurrency".
- Fairneris (S3) vs REF-C02. S3: "IronFleet supports a limited class of liveness properties for a limited class of programs by reducing liveness to termination of sequential programs. In IronFleet each node of the distributed system must be implemented as a simple single-threaded event loop—liveness is proved by showing that all event handlers terminate." This is narrower than REF-C02's account of a protocol-level TLA-embedded liveness proof under the `LivenessAssumptions` conjunction.
- Grove (S2) vs S1. S2: "IronFleet also includes a proof of liveness, though it does not handle thread concurrency, leases, crashes, or reconfiguration". S2 agrees with REF-C02 that liveness is proved, and with S1 that thread concurrency is absent.
- Aneris paper vs code. S4 (ESOP 2020): "Aneris provides no guarantee of delivery or ordering although we assume duplicate protection." The current semantics (`lang.v:1032`, `MessageDuplicateStep`) permits duplication. Both are cited only for their own versions.

## 14. Sources
| ID | Title | URL | P/S | Depth | Notes |
|---|---|---|---|---|---|
| S1 | Timany et al., "Trillium: Higher-Order Concurrent and Distributed Separation Logic for Intensional Refinement", POPL 2024 | https://iris-project.org/pdfs/2024-popl-trillium.pdf | P | [SECT] | Whole PDF retrieved. Read: abstract, Aneris/TLA+ case studies, §6 Related Work (IronFleet) |
| S2 | Sharma et al., "Grove: a Separation-Logic Library for Verifying Distributed Systems" (extended), SOSP 2023 | https://pdos.csail.mit.edu/6.824/papers/grove.pdf | P | [SECT] | Whole PDF retrieved. Read: abstract, intro limitation, execution model (crashes, network, time), §6 Implementation table, Related Work |
| S3 | Hinrichsen, Stefanesco, Birkedal, Timany, "A Higher-Order Separation Logic for Verifying Liveness-Preserving Refinements of Distributed Systems", HAL hal-05647709 (2026-06-07) | https://hal.science/hal-05647709/document | P | [SECT] | Whole preprint retrieved. Read: abstract, intro and contributions, network fairness, Related Work (IronFleet), future work |
| S4 | Krogh-Jespersen et al., "Aneris: A Mechanised Logic for Modular Reasoning about Distributed Systems", ESOP 2020 | https://iris-project.org/pdfs/2020-esop-aneris-final.pdf | P | [SECT] | Whole PDF retrieved. Read: abstract, network semantics assumptions |
| S5 | logsem/aneris @ 59ae379 | https://github.com/logsem/aneris/tree/59ae37946838df77fb2c573803b68462a3af9d0f | P | [CODE] | `aneris_lang/lang.v`, `adequacy.v`, `examples/consensus/*`, `examples/transaction_commit/*`, README |
| S6 | logsem/trillium @ 43d0cf4 | https://github.com/logsem/trillium/tree/43d0cf4021cc1d08da008c5aa104aa6f5ea75f17 | P | [CODE] | `trillium/program_logic/traces.v`, `trillium/prelude/classical.v`, `fairis/fuel_termination.v` |
| S7 | mit-pdos/perennial tag sosp2023 @ 820a2a9 | https://github.com/mit-pdos/perennial/tree/820a2a9ab088d298f9854961c55c010d6ea1858d | P | [CODE] | `src/goose_lang/ffi/grove_ffi.v`, `grove_ffi_adequacy.v`, `src/program_proof/simplepb/apps/closed_proof.v`, Goose `bank.v` |

Access date for all: 2026-11-07.

## 15. Unverified leads (excluded from evidence)
- Fairneris mechanisation matching the 2026 preprint: S3 cites an anonymised Rocq artifact. The public branches `fairneris` (`aef7b2a`) and `fairneris-stenning` (`89a740d`) of logsem/aneris are 2023 work in progress and were only listed, not inspected.
- Other liveness-related branches of logsem/aneris (`better-liveness`, `fair_live_refactoring`, `lmfair_toplevel`) were not inspected.
- Perennial default branch `lean` @ `84d5380`: README statements on `goose_dist_adequacy`; owned by REF-L22.
- Perennial Rocq `master` @ `c3b7ecf` (post-SOSP Grove changes) was not inspected.
- `Print Assumptions` output for `closed_bank` was not obtained (no build).
