# REF-C03: Verdi Raft (Coq)

Kind: comparator (non-Lean). Access date for all sources: 2026-11-07.

## 1. Identity
| Field | Value |
|---|---|
| Subject (protocol or algorithm) | Raft consensus protocol; replicated key-value store `vard` built on it |
| Spec it targets | Raft as described in Ongaro and Ousterhout's paper, "including all the invariants from the original Raft paper" [C03-S1]; correctness stated as linearizability against a single-node state machine [C03-S1, C03-S2]. No RFC. |
| Organisation / authors | University of Washington PLSE: Woos, Wilcox, Anton, Tatlock, Ernst, Anderson (CPP 2016) [C03-S1]; Wilcox, Woos, Panchekha, Tatlock, Wang, Ernst, Anderson (PLDI 2015) [C03-S2]; repo maintainers incl. Palmskog (coq-community templates) [C03-S3] |
| Primary URLs | https://homes.cs.washington.edu/~mernst/pubs/raft-proof-cpp2016.pdf ; https://homes.cs.washington.edu/~mernst/pubs/verify-distsystem-pldi2015.pdf ; https://github.com/uwplse/verdi-raft |
| Repo, commit SHA, last commit date, licence | uwplse/verdi-raft `a3375e867326a82225e724cc1a7b4758b029376f`, 2023-12-08, BSD 2-Clause. uwplse/verdi `7e1641b758d8f0032aa7de63620bc115330e0a6b`, 2026-01-27, BSD 2-Clause. |
| Tool and version | Coq (opam constraint `"coq" {>= "8.14"}`, "Compatible Coq versions: 8.14 or later"); deps `coq-verdi`, `coq-struct-tact`, `coq-cheerios` all `{= "dev"}`; extraction to OCaml (4.02.3 or later) with `verdi-runtime` shim [C03-S3]. |
| Maturity | Research; not production. Repo last commit 2023-12.|
| AI provenance | none-found. Basis: GitHub API, checked 2026-11-07. uwplse/verdi-raft and uwplse/verdi have no AGENTS.md/CLAUDE.md/.cursor/.claude files and 0/100 recent commits with AI co-author trailers. |
| Evidence level | static-inspected (Coq sources read at pinned commit; not rebuilt); performance: author-reported |

## 2. Claim as stated by the authors (verbatim)
- "We present the first formal verification of state machine safety for the Raft consensus protocol, a critical component of many distributed systems. We connected our proof to previous work to establish an end-to-end guarantee that our implementation provides linearizable state machine replication. This proof required iteratively discovering and proving 90 system invariants. Our verified implementation is extracted to OCaml and runs on real networks." [C03-S1]
- "we present the first mechanically checked proof of linearizability of the Raft state machine replication algorithm, as well as verified implementations of a primary-backup replication system and a key-value store. These verified systems provide similar performance to unverified equivalents." [C03-S2]
- README: "Verdi Raft is a verified implementation of Raft in Coq, constructed using the Verdi framework. Included is a verified fault-tolerant key-value store using Raft." [C03-S3]

## 3. What is actually formalised
Modelling style: each node is a set of Coq event handlers (input, message, timeout) over a node state; a network semantics (an inductive step relation) encodes the fault model (reordering, drop, duplication, crash); a verified system transformer (VST) lifts a single-node state machine into a replicated one; correctness is a trace relation [C03-S2]. Implementation and model are the same Coq functions.

Top-level theorem (`theories/RaftProofs/EndToEndLinearizability.v:471-478`, commit a3375e8):
```coq
  Theorem raft_linearizable :
    forall failed net tr,
      input_correct tr ->
      step_failure_star (params := failure_params) step_failure_init (failed, net) tr ->
      exists l tr1 st,
        equivalent _ (import tr) l /\
        exported (get_input tr) (get_output tr) l tr1 /\
        step_1_star init st tr1.
```
i.e. every trace of Raft under the crash/failure network semantics, with well-formed inputs, is equivalent to some linearization `l` that the single-node state machine can produce. Section `Context`s: `orig_base_params`, `one_node_params`, `raft_params` (the replicated application is generic). Concrete instantiation for vard: `theories/Systems/VarDRaftCorrect.v:21` `Theorem vard_raft_linearizable`. Key invariant: `theories/RaftProofs/StateMachineSafetyProof.v:3134` `Theorem state_machine_safety_invariant`. A script `script/assumptions.v` runs `Print Assumptions raft_linearizable.` [C03-S3].

Environment/adversary: "machines may crash at arbitrary points and networks may reorder, drop, or duplicate packets" [C03-S2]; "none of Verdi's network semantics currently capture Byzantine fault models" [C03-S2].

## 3a. Predicate semantics

Headline theorem, `theories/Raft/EndToEndLinearizability.v:471-478` @ a3375e8 [C03-S3]:
```
  Theorem raft_linearizable :
    forall failed net tr,
      input_correct tr ->
      step_failure_star (params := failure_params) step_failure_init (failed, net) tr ->
      exists l tr1 st,
        equivalent _ (import tr) l /\
        exported (get_input tr) (get_output tr) l tr1 /\
        step_1_star init st tr1.
```
(Same statement as `raft_linearizable'` in `theories/Raft/RaftLinearizableProofs.v:1056-1063`.)

Conclusion predicates:
- `equivalent`, `theories/Raft/Linearizability.v:257-261`:
```
  Definition equivalent (l : list op) (ir : list IR) : Prop :=
    good_trace ir /\
    exists ir',
      acknowledge_all_ops l ir' /\
      IR_equivalent ir' ir.
```
  Here `IR` is `IRI`/`IRO`/`IRU` (invoke / return / unacknowledged, lines 17-20); `acknowledge_all_ops` (line 38 onward) adds `IRU` for operations without a response; `IR_equivalent` (lines 119-127) allows only swaps that preserve real-time order; `good_trace` is at lines 249-255.
- `exported`, `RaftLinearizableProofs.v:43-54`:
```
  | exported_IO : forall k i o l tr,
                    env_i k = Some i ->
                    env_o k = Some o ->
                    exported env_i env_o l tr ->
                    exported env_i env_o (IRI k :: IRO k :: l) ((i, o) :: tr)
  | exported_IU : forall k i o l tr,
                    env_i k = Some i ->
                    exported env_i env_o l tr ->
                    exported env_i env_o (IRI k :: IRU k :: l) ((i, o) :: tr).
```
- `step_1_star init st tr1`: the single-node (unreplicated) state machine, run from `init`, produces trace `tr1`.

Strongest premises: `input_correct tr` (`RaftLinearizableProofs.v:994-998`): any two inputs with the same client and request id are the same input. `step_failure_star`: the execution is in Verdi's network semantics with crash failures (packets may be dropped, duplicated and reordered; a node can crash and restart per `failure_params`) [C03-S2].

Plain language. Guaranteed: every trace of the replicated system under that network/failure model can be reordered, respecting real-time order, into a sequential execution of the single-node state machine. For acknowledged operations, the outputs match the clients' observed outputs. For unacknowledged operations (`exported_IU`), the output `o` is unconstrained. Not guaranteed: that any request is ever answered (no liveness); behaviour if clients reuse an id for a different input (excluded by `input_correct`); and correctness of the OCaml shim, extraction, serialisation or the OS network stack (trusted, §6). Byzantine faults are outside the model.

## 4. Property classes covered
| Class | Status | Evidence |
|---|---|---|
| Functional correctness | yes (linearizability of replicated state machine) | `raft_linearizable` [C03-S3] |
| Safety invariant | yes (state machine safety, 90 invariants) | [C03-S1] |
| Liveness | no: "we have only verified Raft's safety properties, leaving liveness for future work" | [C03-S1] |
| Refinement | yes, in the form of trace equivalence to a single-node system via VST | [C03-S2] |
| Symbolic security | no | |
| Computational security | no | |
| Parsing | partial: later variants `vard-serialized` use Cheerios verified serialization; base `vard` shim marshalling is unverified OCaml | [C03-S3, C03-S4] |
| Constant-time | no | |

## 5. Link to implementation code
Mechanism: Coq extraction to OCaml, linked with a trusted OCaml shim. "a Verdi programmer invokes Coq's built-in extraction mechanism to generate OCaml code from the Coq implementation, compile it with the OCaml compiler, and link it with a Verdi shim. The shim is written in OCaml; it implements network primitives" [C03-S2]. Extraction file `extraction/vard/coq/ExtractVarDRaft.v` imports `ExtrOcamlBasic ExtrOcamlNatInt ExtrOcamlString` and Verdi's `ExtrOcamlNatIntExt`, `ExtrOcamlFinInt` (mapping Coq `nat` to OCaml native `int`) [C03-S3]. Unverified OCaml outside generated code in `extraction/`: 1,419 lines (`.ml`, excluding `coq/` subdirectories) [C03-S3]. Guarantee: the extracted handlers are the verified ones; shim, extraction, and network-semantics match are trusted. Not applicable to existing non-Coq code.

## 6. Trusted computing base and soundness hygiene
Static counts (`rg`), at commits above [C03-S3]:

| Repo | `.v` LOC | `Admitted` | `admit.` | `Axiom` | `Parameter` |
|---|---|---|---|---|---|
| verdi-raft | 50,190 | 0 | 0 | 0 | 0 |
| verdi | 16,647 | 0 | 0 | 0 | 38 (all inside module types: `theories/Lib/FMapVeryWeak.v`, `theories/Core/NameOverlay.v`, i.e. interface signatures, not global axioms) |

`Print Assumptions` output not obtained (not built). Author-stated TCB: "the specifications of verified applications, the assumption that Verdi's network semantics match the physical network, the Verdi shim, Coq's proof checker and OCaml code extractor, and the OCaml compiler and runtime" [C03-S2]. Extraction maps (`ExtrOcamlNatInt`) are also trusted (static observation from imports).

## 7. Scale, effort, automation
- CPP 2016: "Our verified implementation of Raft in Coq consists 530 lines of code and 50,000 lines of proof, excluding code from the core Verdi framework"; earlier linearizability proof "about 5000 lines and assumed several nontrivial invariants"; new invariant proofs "about 45000 additional lines"; "90 system invariants"; methodology developed "over a period of roughly 18 months" [C03-S1].
- PLDI 2015 Table 2 (lines incl. blanks/comments): Raft (Linearizability) spec 170 / impl 520 / proof 4,144; Verdi framework spec 148 / impl 220 / proof 2,364 [C03-S2].
- Performance (PLDI 2015 Table 3): etcd 38.9 req/s, get 205 ms, put 198 ms; vard 34.3 req/s, get 232 ms, put 232 ms [C03-S2].
- Automation: interactive Coq with structural tactics, custom induction principles ("planning for change"); no SMT [C03-S1].

## 8. Limitations stated by the authors (verbatim)
- "To date, we have only verified Raft's safety properties, leaving liveness for future work." [C03-S1]
- "It does not support extensions to Raft which are useful in practice, such as dynamic reconfiguration and log compaction. It also includes more data on Acknowledgment messages than is necessary. These limitations are not fundamental, but addressing them would increase the proof burden." [C03-S1]
- "Verdi currently supports verifying safety properties, but not liveness properties, and none of Verdi's network semantics currently capture Byzantine fault models." [C03-S2]
- "vard is not feature complete with respect to etcd, which uses different internal data structures and a more complex network protocol." [C03-S2]
- "Before we applied our recommendations ... we spent the majority of our time reworking proofs in response to changes." [C03-S1]

## 9. Independent evidence
- Bugs found by verification: "servers could delete committed entries when a complex sequence of failures occurred. Such a sequence is unlikely to arise in regular testing" (in their own implementation) [C03-S2]; a client-latency "performance bug" fixed after the proof, with proof repair [C03-S1].
- Independent study (Fonseca et al., EuroSys 2017) found Verdi bugs V1-V8, all outside the verified Coq code: e.g. "V1: Incorrect unmarshaling of client requests throws exceptions" (shim assumed `recv` returns a full request), "V2: Incorrect marshaling enables command injection", "V3: Incomplete log causes crash during recovery", "V4: Crash during snapshot update causes loss of data", "V6: Large packets cause server crashes", "V8: Lagging follower causes stack overflow on leader" (shim runtime); "No protocol bugs were found in the verified systems" [C03-S4].
- Summary row in [C03-S4]: Verdi Raft "54K lines of Coq/OCaml".

## 10. Reproducibility
Not built: sandbox disk limit (instruction from /root: no Coq/opam installs). Commands that would be used (README): `opam repo add coq-extra-dev https://coq.inria.fr/opam/extra-dev && opam install coq-struct-tact coq-cheerios coq-verdi && make` then `coqc` of `script/assumptions.v` for `Print Assumptions raft_linearizable`, and `make vard`. Static inspection only; clones deleted after extraction.

## 11. Generalisation
Pattern: "implementation as proof-assistant functions + network semantics as inductive step relation + verified system transformers + extraction". To apply to an arbitrary codebase: rewrite the protocol logic as pure handlers in the proof assistant; supply a trusted shim for I/O, timers, marshalling and persistence; choose a fault model. Case-specific preconditions: non-Byzantine faults; safety-only (no fairness or temporal logic in Verdi); extraction semantics trusted; shim correctness unproven (the main source of bugs per [C03-S4]).

## 12. Bearing on sub-claims (facts only, no verdict)
| Sub-claim | Fact from this reference | Source ID |
|---|---|---|
| H1 | A dependently typed proof assistant (Coq) expressed Raft, a fault-model network semantics and linearizability; liveness not expressed. | C03-S1, C03-S2 |
| H2 | ~50K lines of Coq proof, 90 invariants, ~18 months methodology development for Raft safety; zero `Admitted`/`Axiom` in repo. | C03-S1, C03-S3 |
| H3 | Link via Coq extraction to OCaml plus a trusted OCaml shim (1,419 lines in `extraction/`). | C03-S2, C03-S3 |
| H4 | Requires rewriting the protocol as Coq handlers; no route for existing code. | C03-S2 |
| H5 | Verified core had no protocol bugs found by independent study; eight bugs found in unverified shim/runtime. | C03-S4 |
| D | Authors use "verified implementation" for extracted Coq handlers with safety proof under a trusted shim and network-semantics assumption. | C03-S1, C03-S2 |

## 13. Conflicts
- Size: CPP 2016 says "530 lines of code and 50,000 lines of proof" [C03-S1]; PLDI 2015 Table 2 lists Raft impl 520, proof 4,144 (linearizability layer only) [C03-S2]; Fonseca et al. list "54K lines of Coq/OCaml" [C03-S4]; repo `.v` total is 50,190 lines [C03-S3]. Different scopes and dates; recorded as given.
- "verified implementation ... runs on real networks" [C03-S1] vs Fonseca et al. Bugs V1-V8 causing crashes, data loss and command injection in the deployed system [C03-S4]; the bugs lie in the trusted shim, which the authors list as TCB [C03-S2].

## 14. Sources
| ID | Title | URL | P/S | Depth | Summary |
|---|---|---|---|---|---|
| C03-S1 | Planning for Change in a Formal Verification of the Raft Consensus Protocol (CPP 2016) | https://homes.cs.washington.edu/~mernst/pubs/raft-proof-cpp2016.pdf | [P] | [FULL] | Raft state machine safety proof in Coq, 90 invariants, 50K proof lines, proof-engineering methodology; liveness future work. |
| C03-S2 | Verdi: A Framework for Implementing and Formally Verifying Distributed Systems (PLDI 2015) | https://homes.cs.washington.edu/~mernst/pubs/verify-distsystem-pldi2015.pdf | [P] | [FULL] | Framework: network semantics, verified system transformers, extraction to OCaml; TCB list; vard vs etcd performance. |
| C03-S3 | uwplse/verdi-raft @ a3375e8 (and uwplse/verdi @ 7e1641b) | https://github.com/uwplse/verdi-raft | [P] | [CODE] | Coq sources; `raft_linearizable` theorem; no Admitted/Axiom; extraction files and OCaml shim. |
| C03-S4 | An Empirical Study on the Correctness of Formally Verified Distributed Systems (EuroSys 2017) | https://www.cs.purdue.edu/homes/pfonseca/papers/eurosys2017-dsbugs.pdf | [P] | [FULL] | Independent study; 16 bugs across IronFleet, Verdi, Chapar, all in specs, tooling or shim; Verdi bugs V1-V8. |

## 15. Unverified leads (excluded from evidence)
- Later Verdi-related work (e.g. Disel, liveness extensions) not read; `coq-inf-seq-ext` dependency of Verdi suggests infinite-sequence reasoning infrastructure exists; not examined.
- `proofalytics/` directory in verdi-raft (proof-time analytics) not examined.
