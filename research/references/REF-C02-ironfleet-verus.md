# REF-C02: IronFleet (Dafny) plus Verus distributed-systems work (IronKV port, Anvil)

Kind: comparator (non-Lean). Access date for all sources: 2026-11-07.

## 1. Identity
| Field | Value |
|---|---|
| Subject (protocol or algorithm) | IronRSL: MultiPaxos-based replicated state machine library; IronKV (IronSHT): lease-based sharded key-value store with reliable-delivery layer; Anvil: Kubernetes controllers (ZooKeeper, RabbitMQ, FluentBit) |
| Spec it targets | Author-written specs: linearizable centralized service spec (IronRSL, 85 SLOC), hash-table spec (IronKV, 34 SLOC) [C02-S1]; Anvil: "eventually stable reconciliation" (ESR) temporal formula [C02-S4]. No RFC. |
| Organisation / authors | IronFleet: Hawblitzel, Howell, Kapritsos, Lorch, Parno, Roberts, Setty, Zill (Microsoft Research) [C02-S1]. Verus: Lattuada, Hance, Bosamiya, ... Hawblitzel, Howell, Lorch, Padon, Parno (MPI-SWS, CMU, MSR, ETH, VMware Research, et al.) [C02-S2]. Anvil: Sun, Ma, Gu, Ma, Chajed, Howell, Lattuada, Padon, Suresh, Szekeres, Xu (UIUC, UW-Madison, VMware Research, Feldera) [C02-S4] |
| Primary URLs | https://www.microsoft.com/en-us/research/wp-content/uploads/2015/10/ironfleet.pdf ; https://www.microsoft.com/en-us/research/wp-content/uploads/2024/09/Verus.pdf ; https://www.usenix.org/system/files/osdi24-sun-xudong.pdf |
| Repo, commit SHA, last commit date, licence | microsoft/Ironclad (`ironfleet/`) `2fe4dcdc323b92e93f759cc3e373521366b7f691`, 2023-06-02, MIT (README: "IronFleet is licensed under the MIT license"). verus-lang/verified-ironkv `7e35ed097c4089b9fbda483ea2629a0033f4d874`, 2026-09-21, MIT. verus-lang/verus `dc1b61c7affc1f947ce0724134bec33656ae0da0`, 2026-11-06, MIT. |
| Tool and version | IronFleet: Dafny v3.4.0 + .NET 6.0 SDK (README of current release; paper used 2015 Dafny compiled to C#). Verus: vstd `0.0.0-2026-11-01-0251`, `rust-toolchain.toml` channel `1.99.1`, Z3 `4.16.0` (`source/tools/get-z3.sh`). |
| Maturity | IronFleet: research prototype, repo inactive since 2023. Verus: actively developed research tool (Distinguished Artifact Award, SOSP'24). verified-ironkv: research port. Anvil controllers: research, "can readily be deployed" [C02-S4]. Not AI-generated. |

## 2. Claim as stated by the authors (verbatim)
- IronFleet: "We describe a methodology for building practical and provably correct distributed systems based on a unique blend of TLA-style state-machine refinement and Hoare-logic verification ... We prove that each obeys a concise safety specification, as well as desirable liveness requirements." [C02-S1]
- IronFleet: "Our proofs reason all the way down to the bytes of the UDP packets sent on the network" and "We provide the first machine-verified liveness proofs of" non-trivial distributed system implementations [C02-S1].
- IronRSL: "We prove complete functional correctness and its key liveness property: if the network is eventually synchronous for a live quorum of replicas, then a client repeatedly submitting a request eventually receives a reply." IronKV: "if the network is fair then the reliable-transmission component eventually delivers each message." [C02-S1]
- Verus: "We evaluate Verus's effectiveness with a wide variety of case-study systems, including distributed systems ... together comprising 6.1K lines of implementation and 31K lines of proof. Verus verifies code 3–61× faster and with less effort than the state of the art." [C02-S2]
- Anvil: "We present Anvil, a framework for developing controller implementations in Rust and verifying that the controllers correctly implement eventually stable reconciliation." [C02-S4]

## 3. What is actually formalised
Layering (IronFleet): trusted high-level centralized spec; untrusted distributed protocol layer (abstract host state machines, TLA-style); implementation layer (imperative Dafny) proved by Floyd-Hoare reasoning to refine the protocol layer; a reduction obligation lets atomic host steps stand for interleaved I/O [C02-S1].

Top-level IronFleet theorem, trusted statement (`ironfleet/src/Dafny/Distributed/Common/Framework/Main.s.dfy:56-61`, commit 2fe4dcd):
```dafny
lemma RefinementProof(config:DS_s.H_s.ConcreteConfiguration, db:seq<DS_State>) returns (sb:seq<ServiceState>)
  requires |db| > 0
  requires DS_Init(db[0], config)
  requires forall i {:trigger DS_Next(db[i], db[i+1])} :: 0 <= i < |db| - 1 ==> DS_Next(db[i], db[i+1])
  ensures  |db| == |sb|
  ensures  Service_Init(sb[0], Collections__Maps2_s.mapdomain(db[0].servers))
  ensures  forall i {:trigger Service_Next(sb[i], sb[i+1])} :: 0 <= i < |sb| - 1 ==> sb[i] == sb[i+1] || Service_Next(sb[i], sb[i+1])
  ensures  forall i :: 0 <= i < |db| ==> Service_Correspondence(db[i].environment.sentPackets, sb[i])
```
Instantiated for IronRSL at `Services/RSL/Main.i.dfy:1029`. IronRSL service spec (`Services/RSL/AbstractService.s.dfy`, header comment): "It provides linearizability of state-machine operations requested by clients ... Note that the specification does not require exactly-once semantics."

Environment: "the network may arbitrarily delay, drop, or duplicate packets. We do assume the network does not tamper with packets, and that the addresses in packet headers are trustworthy" [C02-S1]. No cryptographic adversary.

Verus IronKV port: verifies only the host program against the protocol-level host description. README: "this repository only verifies the 'host program' from IronFleet ... This repository doesn't replicate that layer of proof" [C02-S3]. Per-step obligation (`ironsht/src/host_impl_t.rs:126-143`, commit 7e35ed0):
```rust
pub open spec fn next_ensures(old_self: Self, old_netc: NetClient, new_self: Self, new_netc: NetClient, rc: (bool, Ghost<EventResults>)) -> bool
{
    let (ok, res) = rc; {
        &&& ok == new_netc.ok()
        &&& ok ==> new_self.invariants(&new_netc.my_end_point())
        &&& ok ==> Self::next(old_self.view(), new_self.view(), res@.ios)
        &&& ok ==> res@.event_seq() == res@.ios
        &&& (ok || res@.sends.len()>0) ==> new_netc.history() == old_netc.history() + res@.event_seq()
        &&& res@.well_typed_events()
    }
}
pub fn next_impl(&mut self, netc: &mut NetClient) -> (rc: (bool, Ghost<EventResults>))
    requires Self::next_requires(*old(self), *old(netc)),
    ensures  Self::next_ensures(*old(self), *old(netc), *final(self), *final(netc), rc),
```
Anvil ESR (eq. 1): "∀d. □(□desire(d) ⇒ ◇□match(d))" and "model ⊨ ∀d. □desire(d) ⇝ □match(d)"; "Verus does not support temporal logic reasoning, so Anvil provides a TLA embedding on top of first-order logic" [C02-S4].

## 4. Property classes covered
| Class | Status | Evidence |
|---|---|---|
| Functional correctness | yes (IronFleet, Verus IronKV host, Anvil controllers) | [C02-S1, C02-S3, C02-S4] |
| Safety invariant | yes | IronFleet refinement to linearizable spec; Anvil RabbitMQ "replicas number never decreases" [C02-S1, C02-S4] |
| Liveness | yes (IronFleet, Anvil); no (Verus IronKV port skips protocol layer) | [C02-S1, C02-S3, C02-S4]; IronFleet liveness contains one live `assume` (section 6) |
| Refinement | yes (impl -> protocol -> spec in IronFleet; host impl -> host protocol in Verus port) | [C02-S1, C02-S3] |
| Symbolic security | no | network assumed non-tampering [C02-S1] |
| Computational security | no | |
| Parsing | yes: marshalling/parsing verified (IronFleet generic library; Verus `Marshallable` trait with proofs) | [C02-S1, C02-S2] |
| Constant-time | no | |

## 5. Link to implementation code
Mechanism: the verification language is the implementation language. IronFleet: Dafny compiled to C# and run on .NET, with a trusted UDP specification ("we extend the language with a trusted UDP specification that exposes ... these Dafny methods invoke the .NET UDP network stack") [C02-S1]. Verus/Anvil: verified Rust; "Verus erases all but Rust content for tools other than the verifier; standard rustc sees only executable code" [C02-S2]. Guarantee: executions of the compiled host loop, under the trusted main loop and I/O spec, refine the spec. Not applicable to existing unannotated code: implementations were written (or ported) for verification.

## 6. Trusted computing base and soundness hygiene
Static counts (`rg`), at commits above:

| Repo | files | LOC | live `assume` | commented `//assume` | `{:axiom}` | `external_body` | `admit` |
|---|---|---|---|---|---|---|---|
| Ironclad/ironfleet (`.dfy`) | 267 | 66,568 | 1 | 5 | 64 (56 in `Common/Native/Io.s.dfy`, 8 in `Services/RSL/AppStateMachine.s.dfy`) | n/a | n/a |
| verified-ironkv (`.rs`) | n/a | 10,870 (2,481 in `*_t.rs` trusted files, 7,639 in `*_v.rs`) | 0 (`assume(`) | 0 | n/a | 25 (all in trusted files: `hashmap_t.rs` 13, `endpoint_hashmap_t.rs` 7, `io_t.rs` 5) | 0 |

The one live assumption is in IronRSL's liveness proof, `Protocol/RSL/LivenessProof/ViewAdvance.i.dfy:321`: `assume TimeNotZeno(timefun);`.

Author-stated TCB: "the spec for each system is trusted, as is the brief main-event loop"; "We assume the correctness of Dafny, the .NET compiler and runtime, and the underlying Windows OS ... We also rely on the correctness of the underlying hardware"; the reduction argument is informal: "A machine-checked proof that this obligation enables reduction is future work; instead, we sketch an informal argument" [C02-S1]. Liveness assumptions: IronRSL "a quorum of replicas run their respective main loops with a minimum frequency, never running out of memory, and the network eventually delivers messages synchronously"; IronKV "each host's main loop executes infinitely often and that the network is fair" [C02-S1]. Verus: "Verus's results depend on the correctness of the top-level specifications, the Verus verifier, the solvers it relies on, and the Rust compiler" [C02-S2]. Anvil: "trusted components, including the model of the environment, the shim layer, trusted external APIs, and the verifier, compiler, and OS" [C02-S4].

## 7. Scale, effort, automation
- IronFleet Figure 12 totals: spec 1,400 / impl 5,114 / proof 39,253 SLOC; verification time 395 min. High-level spec IronRSL 85, IronKV 34. Proof:code at implementation layer "3.6 to 1". Effort "approximately 3.7 person-years". Build: "a full integration build done serially requires approximately six hours" [C02-S1].
- Performance: "IronRSL's peak throughput is within 2.4× of the baseline" (EPaxos MultiPaxos Go); "IronKV's performance is competitive with that of Redis" [C02-S1].
- Verus Figure 9: IronKV→Verus trusted 1,613 / proof 4,509 / code 1,533, P/C 2.9, 41 s (1 core), 18 s (8 cores); IronKV→Dafny trusted 1,205 / proof 8,070 / code 1,923, P/C 4.2, 445 s / 201 s. Verus total across five case studies: 4,692 trusted / 31,231 proof / 6,119 code [C02-S2].
- Anvil Table 1 totals: trusted 2,326 / exec 3,940 / proof 23,975; 1,208 s (401 s parallel); "around 2.5 person-months" per controller; P/C 4.5-7.4; framework "5353 lines of reusable lemmas and 7817 lines of trusted code" [C02-S4].
- Automation: Z3 via Dafny/Boogie or Verus; IronFleet library of "40 fundamental TLA rules"; Verus EPR mode for the IronKV delegation map [C02-S1, C02-S2].

## 8. Limitations stated by the authors (verbatim)
- "IronFleet's correctness is not absolute; it relies on several assumptions (§2.5)." [C02-S1]
- "Most developers would prefer to use a language like C++, so enabling that is an important topic of future work." [C02-S1]
- "aiming for full verification makes it challenging to reuse existing libraries, e.g., for optimized packet serialization." [C02-S1]
- "they still lack many standard features offered by the unverified baselines ... post-crash recovery requires reasoning about the effects of machine crashes that wipe memory but not disk." [C02-S1]
- "In future work, we aim to mechanically verify our reduction argument and prove that our implementation run its main loop in bounded time, never exhausts memory, and never reaches its overflow-prevention limit" [C02-S1]
- Verus IronKV: "We skip the protocol level, since Dafny and Verus share similar mathematical modeling tools and are likely to admit very similar proofs." and "The trusted column is noisy because of how we reinterpreted the original protocol layer as our spec." [C02-S2]; README of port: "IronFleet gives us netc.IsOpen(), but it seems to be rotted, so we're ignoring it" (code comment, `host_impl_t.rs:107`) [C02-S3].
- Anvil: "Note that ESR does not preclude all possible controller bugs. For example, ESR may not rule out all potential safety violations." and "If the unverified controllers are custom controllers not modeled in Anvil, Anvil cannot reason about their interactions with verified controllers" [C02-S4].

## 9. Independent evidence
- Anvil: functional tests (Acto) "found a bug in the ZooKeeper controller ... caused by an incomplete specification of a trusted ZooKeeper API"; crash tests (Sieve) found none; "The verification process exposed deep bugs in both our early implementations and unverified reference controllers" [C02-S4].
- Verus IronKV port confirmed fidelity by benchmarking against IronFleet's harness: "The Verus version performs comparably to the IronFleet original" [C02-S2].
- Verus paper reports external use: "other research groups have already used Verus to produce verified cluster-management controllers, a verified microkernel, a security module for confidential VMs, and to verify LLM-produced code" [C02-S2]. Verus persistent log "integrated into a production codebase" [C02-S2].
- Independent study (Fonseca et al., EuroSys 2017, University of Washington) of IronFleet, Verdi and Chapar: "we found a total of 16 bugs"; for IronFleet, Bug I1 "Incomplete high-level specification prevents verification of exactly-once semantics" (a 7-line patch disabling deduplication "still verified"), Bug I2 "Prover crash causes incorrect validation" (NuBuild caused the verifier "to falsely report that any program passed verification checks, including programs that asserted false"), Bugs I3-I4 in verification tooling; "we found no shim layer bugs in IronFleet, which is the system studied with fewest unverified components"; "No protocol bugs were found in the verified systems" [C02-S7]. The current IronRSL spec header states "the specification does not require exactly-once semantics" [C02-S5].

## 10. Reproducibility
Not built: sandbox disk limit (instruction from /root: no Dafny/.NET/Verus/Rust toolchain installs). Commands that would be used: IronFleet `scons --dafny-path=<dafny>` (per `ironfleet/README.md`); verified-ironkv `scons --verus-path=<path_to_verus>` (per README). Static inspection only; clones deleted after extraction.

## 11. Generalisation
Pattern: three-layer refinement (trusted centralized spec, protocol state machine, imperative host implementation) plus a reduction obligation encoded in the I/O interface, with a temporal-logic embedding in a first-order SMT verifier for liveness. To apply to an arbitrary codebase: the implementation must be written in the verifier's language (Dafny→C#, or Verus Rust); the I/O boundary needs a trusted spec; each host must follow a single event loop with receive-before-send discipline. Case-specific preconditions: non-Byzantine, non-tampering network; author-chosen simple specs; no crash recovery; reliance on SMT automation.

## 12. Bearing on sub-claims (facts only, no verdict)
| Sub-claim | Fact from this reference | Source ID |
|---|---|---|
| H1 | SMT-based verifiers without native temporal logic were extended with TLA embeddings to express liveness (IronFleet, Anvil). | C02-S1, C02-S4 |
| H2 | Machine-checked safety and liveness of Paxos-based RSM implementation at ~39K proof SLOC, 3.7 person-years; one `assume TimeNotZeno` and an informal reduction argument remain. | C02-S1, C02-S5 |
| H3 | Link is "verifier language is the implementation" (Dafny→C#, Verus→Rust via rustc); I/O, main loop, specs trusted. | C02-S1, C02-S2 |
| H4 | Requires writing code in Dafny or Verus-annotated Rust; authors note developers "would prefer ... C++". | C02-S1 |
| H5 | Verus IronKV port verifies the host implementation against the protocol-level host description without re-doing the protocol-level proof. | C02-S2, C02-S3 |
| H5 (cont.) | Independent study found IronFleet bugs only in high-level spec and verification tooling, none in verified protocol code. | C02-S7 |
| D | "verify" = machine-checked refinement and temporal properties under stated assumptions; Anvil adds testing (Acto, Sieve) as a separate check. | C02-S1, C02-S4 |

## 13. Conflicts
- IronFleet claims "complete functional correctness" and machine-verified liveness [C02-S1]; the same paper states the reduction argument is informal, and the code base contains a live `assume TimeNotZeno(timefun);` in the liveness proof (`ViewAdvance.i.dfy:321`) [C02-S5]. Recorded, not resolved.
- Verus paper reports IronKV→Dafny proof at 8,070 lines and trusted 1,205 [C02-S2]; IronFleet Figure 12 reports IronKV implementation-layer proof 2,937, impl 1,340, and spec 34 SLOC [C02-S1]. Verus paper notes counts are "noisy because of how we reinterpreted the original protocol layer as our spec"; different counting scopes.

## 14. Sources
| ID | Title | URL | P/S | Depth | Summary |
|---|---|---|---|---|---|
| C02-S1 | IronFleet: Proving Practical Distributed Systems Correct (SOSP 2015) | https://www.microsoft.com/en-us/research/wp-content/uploads/2015/10/ironfleet.pdf | P | FULL | Dafny methodology combining TLA refinement and Hoare logic; IronRSL and IronKV with safety and liveness; effort, size, assumptions. |
| C02-S2 | Verus: A Practical Foundation for Systems Verification (SOSP 2024) | https://www.microsoft.com/en-us/research/wp-content/uploads/2024/09/Verus.pdf | P | FULL | Rust verifier; five case studies incl. IronKV host port; line counts and verification times vs Dafny. |
| C02-S3 | verus-lang/verified-ironkv @ 7e35ed0 | https://github.com/verus-lang/verified-ironkv | P | CODE | Verus port of IronSHT host; no assume/admit; 25 external_body in trusted files; protocol layer not replicated. |
| C02-S4 | Anvil: Verifying Liveness of Cluster Management Controllers (OSDI 2024) | https://www.usenix.org/system/files/osdi24-sun-xudong.pdf | P | FULL | Verus-based framework with TLA embedding; ESR liveness of three Kubernetes controllers; trusted models; bug found in trusted API spec. |
| C02-S5 | microsoft/Ironclad (ironfleet) @ 2fe4dcd | https://github.com/microsoft/Ironclad/tree/main/ironfleet | P | CODE | Dafny sources; top-level RefinementProof lemma; 1 live assume, 64 axioms in trusted native/IO specs. |
| C02-S7 | An Empirical Study on the Correctness of Formally Verified Distributed Systems (Fonseca, Zhang, Wang, Krishnamurthy; EuroSys 2017) | https://www.cs.purdue.edu/homes/pfonseca/papers/eurosys2017-dsbugs.pdf | P | FULL | Independent bug study of IronFleet, Verdi, Chapar: 16 bugs in specs, tooling and shim layers; none in verified protocol code. |
| C02-S6 | verus-lang/verus @ dc1b61c | https://github.com/verus-lang/verus | P | CODE | Verifier source; Rust toolchain and Z3 pins recorded. |

## 15. Unverified leads (excluded from evidence)
- IronFleet CACM 2017 version ("Proving Safety and Liveness of Practical Distributed Systems") listed in repo README; not read.
- Verus SOSP'24 artifact case-studies page (verus-lang.github.io/paper-sosp24-artifact) seen only as search snippet.
- Verus-based verified microkernel (Atmosphere) and confidential-VM security module cited in [C02-S2]; not read.
