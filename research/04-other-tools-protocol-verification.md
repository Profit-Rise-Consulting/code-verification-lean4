# 04 - Other tools: how they verify protocols and their implementations (comparators)

Phase 1 evidence file. No analysis and no conclusions about Lean 4. Facts, quotes and sources only.

- Access date for every source below: **2026-11-07** (all retrieved on that date).
- Retrieval mode: **FETCH** = page body read in full via fetch; **SNIP** = only the search-result excerpt was read (claims from SNIP sources are limited to what the excerpt showed).
- Tag: **P** = primary (tool's own docs/site, paper, project repo, standards body, author page). **S** = secondary (blog, press, aggregator, third-party summary, student/hobby project).
- Note on search-engine "Date:" fields: many show 2026 crawl dates for papers published years earlier. They are not publication dates and are not used as such here.
- Inline cites use `[S#]` and resolve to the Source Register at the end of the file.

---

## 0. One-table overview (what each tool's input is checked against)

"Target" = the artifact the tool's proof/check is about, as stated in the sources. Cells cite the evidence in the per-tool sections.

| Tool | Target of the check | Stated implementation link | Unbounded? | Evidence |
|---|---|---|---|---|
| Tamarin | Protocol model (multiset rewriting) in symbolic model | None in the tool; separate research frameworks extract I/O specs for Go/Java code (Arquint et al.) | Yes (unbounded sessions); may not terminate | S1, S2, S6, S4, S82 |
| ProVerif | Protocol model (applied pi calculus / Horn clauses), symbolic | None in the tool; hax/Rust-to-ProVerif extraction cited by third parties | Yes; may not terminate, may give false attacks | S9, S10, S11 |
| CryptoVerif | Protocol model, computational model (game sequences) | cv2fstar compiles CryptoVerif models to F* specs (thesis) | Polynomial in security parameter | S13, S16 |
| Verifpal | Protocol model, symbolic | None; docs say a pass does not show an implementation is secure | No: bounded sessions, bounded search | S18, S19, S21 |
| TLA+ / TLC / Apalache | Spec (state machine). TLC explicit-state, Apalache bounded/inductive | Trace validation / test generation as add-on techniques (MongoDB, etcd raft, CCF) | TLC: finite instances; Apalache: bounded length or inductive invariant | S23, S24, S26-S29 |
| Ivy / mypyvy | Ivy: inductive invariants for protocol models; also compositional spec-based testing of real implementations | Network-centric compositional testing of QUIC implementations (testing, not proof) | Decidable-fragment proofs; testing is bounded runs | S31-S33, S35, S36 |
| Coq/Rocq: Verdi, Velisarios | Implementation written in Coq, extracted to OCaml | Extraction (OCaml); shim layer unverified | Yes (proof) | S37, S39, S48 |
| Coq/Rocq: VST, Fiat-Crypto | C code / crypto arithmetic code generator | Proof about C program (VST); proof about generator output (Fiat) | Yes | S40, S42, S43, S44 |
| Coq/Rocq + Iris: Aneris/Trillium | Distributed programs in AnerisLang (OCaml-like) | Refinement to TLA+ models (Trillium) | Yes | S45-S47 |
| Isabelle/HOL | Protocol models (inductive method); refinement frameworks (Igloo) | Igloo links Isabelle models to code verified in VeriFast/Nagini | Yes | S49-S53 |
| F* / Project Everest | Implementation code (Low*) with proofs; extracted to C | Code is the artifact; TLS 1.3 handshake implementation proof not completed | Yes | S56, S57 |
| Dafny / IronFleet | Implementation code refining a protocol layer refining a spec | Three-layer refinement, all in Dafny | Yes (proof) | S62, S63 |
| Verus | Rust implementation code vs specs | Code is the artifact | Yes (proof, SMT-based) | S65, S66 |
| Kani | Rust implementation harnesses | Code is the artifact | No: bounded (loop unwinding) | S70-S73 |
| P | Model (communicating state machines); PObserve checks logs against P specs | PObserve validates service logs against P specs | Model checking, bounded exploration | S75-S77 |
| Stateright | Rust actor model checked; same actors can run on UDP | Same code is model checked and run | Bounded/finite state spaces | S78-S80 |

---

## 1. Tamarin

**What it verifies.** Protocol models in the symbolic (Dolev-Yao) model; it reports a proof for an unbounded number of sessions or a counterexample.
- "If the tool's automated proof search terminates, it returns either a proof of correctness (for an unbounded number of role instances and fresh values) or a counterexample, representing an attack that violates the stated property." [S2, P, FETCH-snippet]
- Core modelling mechanism is multiset rewrite rules; process-calculus input is translated to rewrite rules. [S3, P]
- Authors' TLS 1.3 talk slides: "We analyse the specification only" and "We assume black-box cryptography". [S4, P, SNIP]

**Case studies.**
- TLS 1.3 draft 10 (NDSS/CCS 2016-17 line) [S4, P] and draft 21 "most comprehensive, faithful, and modular symbolic model of the TLS 1.3 draft 21 release candidate" [S5, P, SNIP].
- 5G AKA: "We provide the first comprehensive formal model of a protocol from the AKA family: 5G AKA ... some critical security goals are not met, except under additional assumptions missing from the standard." [S6, P, SNIP]. Cremers' page: vulnerability found in the Feb 2018 draft of 3GPP TS 33.501 v0.7.0; the page "does not state that an implementation was analysed". [S7, P, FETCH]
- Also listed by the project: Noise, EMV, Apple iMessage. [S3, P]
- WireGuard: Tamarin symbolic model (Donenfeld/Milner) with properties listed on the WireGuard site: correctness, key agreement, KCI resistance, UKS resistance, key secrecy, forward secrecy, session uniqueness, identity hiding. [S15, P, SNIP]
- WireGuard also in SAPIC+ with ProVerif and Tamarin: ProVerif agreement/secrecy/PFS ~15 min each, anonymity ~2-9 h; "we verify the CNFs with Tamarin due to its more complete Diffie-Hellman model". [S84, P, SNIP]
- Implementation link (separate research, not Tamarin itself): Arquint et al. generate I/O specifications from a Tamarin model and verify Go code (Gobra) against them; see Section 17. [S82]

**Documented limitations.**
- "Tamarin operates in the symbolic model and thus can only capture attacks within that model, and given a certain equational theory. Currently, apart from the builtins, only subterm-convergent theories are supported. The underlying verification problems are undecidable in general, so Tamarin is not guaranteed to terminate." [S1, P]
- "the observational equivalence mode currently only (soundly) approximates observational equivalence by requiring a strict one-to-one mapping between rules ... restrictions in this mode is rather limited." [S1, P]
- Non-subterm-convergent equations "can result in infinite loops or incorrect results." [S8, P, SNIP]
- Tutorial slides: "Limited equational theories, e.g., MQV style exponentiation tricky: we miss Kaliski's UKS attack on MQV." [S9b, P, SNIP]
- Verifpal's comparison page (a competitor's statement, tag S): Tamarin "does not guarantee a finished proof"; untagged NSLPK3 needs a `reuse` annotation. [S18, P for Verifpal, secondary for claims about Tamarin]

---

## 2. ProVerif

**What it verifies.** Symbolic Dolev-Yao models, reachability and equivalence properties, via Horn-clause resolution.
- "ProVerif has been used to verify and improve Signal, TLS, avionic protocols, internet voting, and much more, in the Dolev-Yao model." [S9, P, SNIP]
- Handles an unbounded number of sessions: "ProVerif does not rely on a bounded-session assumption nor on a general decidability result." [S11, P, SNIP]

**Case studies.** TLS 1.3 (drafts 18 and 20 per a third-party paper [S12b, S]), Signal [S12, P, SNIP: "large parts of industrial-strength protocols, such as TLS, Signal, JFK, and Web Services Security"], Noise (via Noise Explorer: generates ProVerif models of Noise patterns; WireGuard site cites Noise Explorer IK) [S15, P, SNIP], WireGuard [S84], QUIC handshake (two academic modelling efforts with SPIN+ProVerif and ProVerif+Verifpal; ProVerif found "A1, A2 and S2 violated" in the second) [S87, S88, P, SNIP].

**Documented limitations.**
- "Limitations: Risk of non-termination. Risk of false attacks. Still limited on the equations it supports and the equivalences it can ..." [S10, P, SNIP, Blanchet slides]
- Approximation: "repetitions of actions are ignored: the clauses can be applied any number of times" and "They can cause (rare) false attacks." [S10, P, SNIP]
- "The tool may fail to terminate or return 'cannot be proved', but when it succeeds in proving a property, the result is sound." [S11, P, SNIP]
- Lecture notes: "may not terminate, may terminate with real or false attack; Proverif's procedure works very well in practice, but offers no guarantee." [S12c, S, SNIP]

---

## 3. CryptoVerif

**What it verifies.** Protocol models in the computational model, proofs by sequences of games; secrecy, correspondences (authentication), indistinguishability.
- "These proofs are valid for a number of sessions polynomial in the security parameter, in the presence of an active adversary." [S13, P, SNIP]
- Output includes "an upper bound of the probability of success of an attack." [S17, P, SNIP]

**Case studies.** TLS 1.3 (S&P 2017), Signal (EuroS&P 2017), WireGuard (EuroS&P 2019; "We analyse the entire WireGuard protocol as it is, including transport data messages, in an ACCE-style model.") [S13, S14, S16b, P, SNIP]. A DY* paper notes that with CryptoVerif "the analysis of Signal has to be limited to just one ratcheting round". [S86, P, SNIP]
- Dynamic key compromise paper: proves forward secrecy w.r.t. PSK compromise in WireGuard "which could not be proved in [39, Section VI]". [S17b, P, SNIP]

**Implementation link.** Lipp's thesis: "there is frequently a gap between a cryptographic security analysis and an implementation that have both been based on a protocol specification: no formal guarantee exists that the two interpretations of the specification match". Contribution: `cv2fstar`, "a compiler from CryptoVerif models to executable F* specifications using the HACL* verified cryptographic library as backend", demonstrated on Needham-Schroeder-Lowe. The abstract names no limitations of cv2fstar and does not mention hacspec or Rust. [S16, P, FETCH]

**Limitations.** Computational-model proofs are valid for polynomial session counts [S13]. Dynamic compromise needed extensions [S17b]. No tool-level limitations page was fetched (gap).

---

## 4. Verifpal

**What it verifies.** Protocol models (its own DSL), symbolic active attacker, with a fixed set of built-in primitives.
- "Verifpal's heuristic search can miss attacks even at the selected session count, so it is incomplete within that bound." [S18, P, SNIP]
- "A pass means it found none within the search limits. Neither establishes that an implementation is secure or that the protocol is secure for an unbounded number of sessions." [S19, P, SNIP]
- Quickstart: "It does not prove that a deployed implementation is secure." [S19b, P, SNIP]
- Model rules "omit byte lengths, timing, implementation bugs and computational probabilities." [S20, P, SNIP]
- The 2026 evaluation paper characterizes it as "a bounded counterexample finder: every reported attack is an explicit protocol execution, but a passing query is not an unbounded proof, and no report is a machine-checked certificate." [S21, P, SNIP]
- The crate page: "Verifpal supports neither observational equivalence nor user-defined equational theories, and it always terminates instead of offering unbounded session replication." [S21b, P, SNIP]
- 2020 paper abstract claimed unbounded sessions and Coq-formalised semantics [S22, P, SNIP]; the 2026 materials describe bounded sessions (default 2 per principal, range 1-16) [S19]. Both statements are recorded as written; they are not reconciled here.

**Case studies.** Signal, Scuttlebutt, TLS 1.3, Telegram, DP-3T; QUIC handshake (via the ProVerif/Verifpal comparison) [S20, S22, S88].

---

## 5. TLA+ / TLC / Apalache

**What it verifies.** A specification (state machine plus temporal properties), not code.
- TLC "is an explicit-state model checker, which means it enumerates all possible states ... state-space needs to be small enough ... Checks executions of any length." [S25, S, SNIP (Quint docs)]
- Apalache: "symbolic bounded model checker"; "Apalache cannot check an arbitrary number of steps - it needs to choose a specific number"; default `--max-steps` 10; temporal properties have partial support. [S25, S, SNIP]
- Apalache assumptions (Inria project report): "all specification parameters are fixed and finite"; "all data structures ... are finite"; "Only finite executions up to a given bound are analysed." [S24, P, SNIP]
- Apalache can check inductive invariants for fixed parameters. [S23b, P, SNIP]
- Apalache vs TLC (Springer 2026 chapter): outperformed TLC on many Paxos/Raft cases; "Apalache could provide results in 8 cases in which TLC could not" and TLC "normally explodes" on Byzantine algorithms. [S23c, P, SNIP]

**Case studies.** Paxos and Raft model checking in the Apalache evaluation [S23, P]. MongoDB RaftMongo.tla; etcd raft TLA+ spec with trace validation [S28]; CCF/Raft trace validation repo [S29].

**Spec-to-implementation bridges (documented experience).**
- MongoDB (VLDB 2020, FETCH): trace-checking MongoDB server vs `RaftMongo.tla` (345 lines). "MBTC was impractical for our system." Prototype 10 weeks, 2 engineers (event tracing 570 lines C++, 3 weeks of spec updates to 252 TLA+ lines, post-processor 484 Python). Only 5 handwritten tests and 1 randomized test could be trace-checked; 1 handwritten test passed, 4 violated the spec. Spec/impl differences: arbiters not modelled, initial-sync nodes counted toward quorum, two leaders briefly possible, term propagation, oplog copying. State space grew from 42,034 to 371,368 states; check time from 2 s to 14 min. [S27, P]
- Same project, test-case generation for Realm Sync OT: 6 array operations, 21 merge rules, ~1,000 lines C++; TLA+ spec 795 lines; 4,913 generated tests; branch coverage 100% vs 21% (36 handwritten tests) and 92% (~8M fuzz executions); found a non-terminating ArraySwap/ArrayMove merge (StackOverflowError) present in the C++. Authors: worked because the code was "a self-contained function with defined inputs and outputs" and the state space was manageable; "convergent but undesirable" behavior would not be caught by the invariant alone. [S27, P]
- MongoDB 2025 blog (FETCH): "The traces never matched our specification"; "10 weeks"; "It's hard to snapshot a multithreaded program's state"; "The implementation must actually conform to the spec." [S26, P (vendor engineering blog)]
- etcd raft: spec model-checked with TLC; "adherence of the implementation to algorithm specification, is fortified through trace validation"; trace points injected into Go code; build tag `with_tla`; `Traceetcdraft.tla` consumes NDJSON. [S28, P]
- Specula (arXiv 2026, SNIP, preprint): "For TLA+ models that describe system implementations, the model must conform to the code. Conformance is commonly checked via trace validation"; "A model can therefore remain inconsistent with the implementation in ways we did not catch". [S30, S]

**Documented limitations.** State-space explosion for TLC; bounded length for Apalache [S25, S24]; separate conformance step needed to connect spec to code, with reported cost and failure cases [S26, S27].

---

## 6. Ivy and mypyvy

**What it verifies.**
- Ivy: protocol models via inductive invariants in decidable fragments; also "compositional specification-based testing" of systems "not written in Ivy". [S33, P, SNIP]
- mypyvy: "takes an input file describing a symbolic transition system and can perform various tasks such as inductive invariant checking and inference". [S35, P]
- Paxos made EPR (Padon et al.): safety of Paxos, Multi-Paxos, Vertical, Fast, Flexible, Stoppable Paxos in a decidable logic, "first mechanized safety proof" for three variants; mechanized also in Isabelle/HOL. [S36, P, SNIP]

**QUIC case study (McMillan & Zuck, SIGCOMM 2019; FETCH).**
- Method: formal Ivy wire specification of QUIC used as an executable monitor and SMT-based randomized tester that takes one role. "our specification is not an abstract implementation of the protocol, but is instead an executable monitor". [S31, P]
- Result: four implementations tested; "27 errors in the server implementations" (13 protocol errors, 12 crashes, 2 failures to make progress); 4 potentially exploitable (includes a Heartbleed-like memory leak); 4 errors attributed partly to standard ambiguities. [S31, P]
- Scope limits: spec omits 0-RTT, version negotiation, Retry, RETIRE_CONNECTION_ID; "captures only safety properties"; "We do not cover any statement in the QUIC document on recovery and congestion control"; encryption/decryption, OS and network are black boxes; TLS handshake events use a real TLS implementation; "a large portion (perhaps most) of the statements in the standard were not usefully testable". Runs limited to 100 protocol events and one connection; generation ~10 Hz. Testing time ~4 weeks of engineer time. [S31, P]
- Presented as testing, not proof: results hold for finite observed executions. [S31, P]
- Ivy QUIC spec is in the Ivy repo (draft-23 based) with Quant and Picoquic configs. [S32, S34, P/S]

**Other.** Ivy reported used for RISC-V memory hierarchy bugs. [S33, P, SNIP]

---

## 7. Coq/Rocq

### 7.1 Verdi, Verdi-Raft, Velisarios
- Verdi: "a framework for implementing and formally verifying distributed systems in Coq"; verified system transformers; "first mechanically checked proof of linearizability of the Raft state machine replication algorithm", plus primary-backup and key-value store; "similar performance to unverified equivalents". [S37, P, SNIP]
- Verdi-Raft: "verified implementation of the Raft distributed consensus protocol in Coq, constructed using the Verdi framework". [S38, P]
- Pipeline: "It invokes Coq to translate its verified code into OCaml, which is then either interpreted ... or compiled into a binary". [S39, P, SNIP]
- Empirical bug study (EuroSys 2017; FETCH): 16 bugs across IronFleet, Verdi, Chapar after >8 months of searching; 11 in shim layers, 2 spec/client-assertion, 3 in verification tooling; "No protocol bugs were found in the verified systems"; shim components "typically only a few hundred lines of source code"; 9 of 11 shim bugs crashed/hung servers. Verdi had 8 shim-layer bugs (partial TCP reads, unescaped marshaling, non-atomic log write, snapshot truncation, transient open error, oversized UDP packets, stack exhaustion in generated OCaml). The unverified comparison sample had 12 protocol bugs. [S39, P]
- Velisarios (Springer): implements and verifies BFT protocols in Coq; says Verdi leaves no gap "between verified and running code" through extraction, and Verdi "do[es] not deal with the full spectrum of arbitrary faults (e.g. malicious faults)". Also "To the best of our knowledge, Ivy does not support faults." [S48, P, SNIP]

### 7.2 VST and OpenSSL/mbedTLS
- VST assures "with machine-checked proofs that the assertions claimed at the top of the toolchain really hold in the machine-language program". [S42, P]
- OpenSSL HMAC: proofs in Coq that the OpenSSL HMAC-SHA256 implementation "correctly implements its FIPS functional specification" and the spec gives the expected crypto properties, combining VST, FCF and CompCert. [S44, P]
- mbedTLS HMAC-DRBG: proved crypto security of the functional spec (hybrid game proof) and that the C implementation correctly implements it; "assuming that the DRBG's initial state is ideally random". [S43, P]
- VST is "entirely implemented in the Coq proof assistant"; user "supplies function specifications, loop invariants". [S43b, P, SNIP]

### 7.3 Fiat Cryptography
- "first verified high-performance implementation of P-256 ... Implementations from our library were included in BoringSSL ... for Chrome, Android, and CloudFlare." [S40, P]
- Press: "about 90 percent of secure Chrome communications currently run their code" (MIT News) [S41, S]; Fiat Cryptography project page says it drives "important parts of (elliptic-curve-based) TLS in both Chrome and Firefox". [S41b, P, SNIP]
- Verifies the code generator (proof about generated code), for arithmetic, not a TLS protocol state machine. [S40, S41b]

### 7.4 Iris, Aneris, Trillium, Grove
- Aneris: "higher-order distributed concurrent separation logic", "mechanized in the Coq proof assistant"; verified a load balancer, two-phase commit with replicated log. [S45, S46, P]
- Aneris applications listed by authors: causally consistent DB, CRDTs, single-decree Paxos, snapshot-isolation database. [S45b, P, SNIP]
- Trillium: shows "refinement relations between distributed systems and TLA+ models"; two-phase commit and single-decree Paxos refine abstract TLA+ specs. [S47, P, SNIP]
- Trillium paper on IronFleet: "does not support node-local concurrency"; liveness argued on paper for simple event-loop programs. [S47, P, SNIP]

---

## 8. Isabelle/HOL

**Protocol-level (spec-side).**
- Paulson inductive method: protocols as inductively defined trace sets, proofs by induction, Isabelle/HOL; "There is no restriction to finite-state systems". [S50, P, SNIP]
- TLS (SSL 3.0 descendant) verified: "six man-weeks of effort and three minutes of processor time". [S49, P, SNIP]
- INRIA critique of the method: "not automatic and requires interaction with the user and also a good expertise (if a proof fails with Isabelle, it is difficult to conclude whether the proof attempt failed or the conjecture to be proved is not valid)". [S51, P (third-party paper), SNIP]
- Cryptography is "idealised" in the inductive method [S54b, S, SNIP (thesis slides)].
- Paxos in Isabelle/HOL: Disk Paxos in ~6,500 lines of proof script (cited in Paxos Made EPR) [S36]; Charron-Bost/Merz verified Paxos at the Heard-Of level [S55, S, SNIP].

**Refinement toward code: Igloo (OOPSLA 2020).**
- "We formalized our framework, Igloo, in Isabelle/HOL. We report on three case studies, a leader election protocol, a replication protocol, and a security protocol ... implement in Java and Python and prove correct using the VeriFast and Nagini tools." [S52, P, SNIP]
- Trillium authors' comparison: Igloo's refinement is "a rather weak refinement relation ... the program and the transition system perform the same I/O operations" (flexibility with respect to program structure). [S47b, P, SNIP]
- Igloo reused in "Formal Verification of a Next-Generation Internet Router" (ETH). [S53b, P, SNIP]
- ETH student-project page: "The Igloo theory ensures that the global properties proved for the models are preserved to the implementation level." [S53, P]

**Other.** Kleppmann's blog models distributed algorithms in Isabelle/HOL (educational) [S54, S]. A SoK on consensus verification lists SPIN/Isabelle for Heard-Of work [S55, S].

---

## 9. F* and Project Everest (miTLS, HACL*, EverParse)

All from the 2026 TOPLAS retrospective (FETCH) unless noted. [S56, P]

**What it verifies.** Implementation code written in F*/Low*, extracted to C/Wasm/Rust by KaRaMeL; SMT (Z3) automation.
- Verified: TLS-1.2 and 1.3 **record layers** (high-performance, with cryptographic security reduction) [S56, S57]; QUIC record layer; HACL* crypto (primarily C); EverParse parsers/serializers (non-malleable), including TLS wire formats (293 datatypes in the 2019 paper) [S58, S56].
- Not completed: "we did not complete verifying our *implementation* of the TLS handshake"; the pen-and-paper handshake/key-schedule proof (state-separating proofs) was "only checked manually" and "we did not manage to translate this pen-and-paper proof to the verified code of miTLS-fstar". Rest of QUIC (CMU, Dafny) "was only proven safe, rather than functionally correct or cryptographically secure." [S56]
- MIT 6.826 lecture (2020): "We generate C and assembly code for TLS-1.3, but the verification is not complete"; record layer complete. [S60, P, SNIP]
- Record-layer paper: reduces record-layer security to cipher assumptions; interoperates with Chrome and Firefox; "part of miTLS ... a larger, partially verified codebase". [S57, P]

**Deployment.** NSS (~17 HACL* algorithms), WireGuard in Linux (EverCrypt Curve25519), Python hash/HMAC, Tezos, ElectionGuard; EverParse in Windows Network Virtualization (>100 message kinds across four protocols, ~5,000 lines of 3D spec to ~30,000 lines C, 5 years in production); miTLS handshake used as default for Microsoft msquic during development, later superseded by Schannel. [S56]

**Documented limitations / TCB.**
- F* "lacks a kernel checker"; trusts VC encoding and SMT solver; **26 soundness bugs** found in F* (3 open); KaRaMeL is unverified OCaml and 6 bugs escaped formalization, internal checks and the C compiler. [S56]
- Scale: ~1.3M lines F* source, ~622,000 proof obligations, ~4,200 files; proof-to-code about 2:1 in executable-focused projects; HACL* 212,000 lines F* to 101,000 lines C; doubly linked list library took one member >6 months. [S56]
- Maintenance: HACL* pinned to old F* in 2017-18; re-upgrade took nearly a year; Z3 pinned to 4.8.5 (2019) until 2024. [S56]
- "No one person on the team was expert in all of these areas." [S56 paraphrase]
- Hindsight: initially judged too costly to relate executable code to an idealized model before proving crypto security; "might have separated concerns". [S56 paraphrase]
- Everest overview: miTLS "powers the primary implementation of the QUIC transport protocol" within Microsoft (blog claim). [S59, P]

---

## 10. Dafny and IronFleet

**What it verifies.** Implementation code in Dafny (compiled to C#), with three layers: high-level spec, abstract distributed protocol layer, implementation layer; refinement proved between layers. [S62, S63, P]
- "the first methodology for automated machine-checked verification of the safety and liveness of non-trivial distributed system implementations" [S62].
- IronRSL (Paxos-based RSM library): 6 spec lines in the protocol-reported table, 2,941 impl, 7,535 proof SLOC, 152 min verify. IronKV: 1,340 impl, 2,937 proof, 42 min. High-level specs 85 and 34 SLOC. Implementation proof:code ratio "3.6 to 1". Effort "approximately 3.7 person-years". [S62, P, FETCH]
- TCB: "The spec for each system is trusted, as is the brief main-event loop"; trusted UDP spec (`Init`, `Send`, `Receive`); network may drop/delay/duplicate but not tamper; correctness of Dafny, .NET, Windows, hardware assumed. [S62]
- Liveness assumptions for IronRSL: quorum, scheduler frequency, bounded delay, burst, clock error, no overflow limit. [S62]
- Limits stated: crash recovery not verified; reduction obligation proof "is future work"; memory exhaustion proof "future work"; focus on newly written code in a verification-friendly language; no Byzantine guarantee stated; performance below unverified baselines; "more up-front development effort". [S62, S63]
- Empirical bug study: IronFleet had 1 specification bug (spec did not require exactly-once semantics; a 7-line patch disabling deduplication still verified) and 3 verification-tool bugs (NuBuild mis-parsing prover crashes, SIGINT handling in Boogie, 32/64-bit z3 mismatch); no protocol bugs. [S39, P, FETCH]
- Source: Ironclad repo README lists Dafny v3.4.0 and .NET 6.0 for the released IronFleet code ("experimental"). [S64, P, SNIP]
- Later work contrasts: IronFleet's refinement technique requires programs "sequential and their structure must mirror the structure of the abstract model". [S89, P, SNIP]

---

## 11. Verus

**What it verifies.** Rust executable code against user specs; SMT-based (Z3) static verification; proofs written in Rust syntax. [S65, S66, P]
- SOSP'24: 6.1K lines of Rust with 31K lines of proof across distributed systems (IronKV-line), OS page table, NUMA replication library, crash-safe storage, memory allocator; "verifies code 3-61x faster and with less effort than the state of the art". [S65, P, SNIP]
- TCB: "Verus's results depend on the correctness of the top-level specifications, the Verus verifier, the solvers it relies on, and the Rust compiler." [S65, P, SNIP]
- Verus's translation to verification conditions "is entirely trusted code"; solvers "not verified or proof-producing" (VerusBelt). [S69, P, SNIP]
- Limits: "At present, we do not intend to support all Rust features and libraries" [S66]; no mutable references in return values; "cannot reason about traditional Rust unsafe code" (OOPSLA'23 extended) [S67, P, SNIP]; unsupported/partial: associated constants, floats, general iterators, `&mut data[i]` (community list) [S66, S68b-S]. RefinedRust authors: "suitable for verifying newly written code ... but not for verifying existing code without significant modifications." [S68, P (third-party paper), SNIP]
- Verus-based Atmosphere microkernel: proof-to-code 7.5:1. [S65b, P, SNIP]

---

## 12. Kani

**What it verifies.** Rust code via proof harnesses; bounded model checking through CBMC.
- "Kani performs bounded model checking. Loops are unwound up to a configurable bound ... Verification results are sound only if all unwinding assertions pass." [S70, P, FETCH]
- Other caveats (all [S70]): CBMC backend bugs; x87 floating-point differences; pointer object-bit limit default 16; unresolved function pointers are over-approximated (spurious failures) and `--restrict-vtable` may cause false negatives.
- "At present, Kani does not support verifying concurrent code." [S72, P, SNIP]
- Does not model Stacked/Tree Borrows, data races (Rust std verification paper; Kani paper); needs trusted stubs for FFI. [S73, S74, P, SNIP]
- "Kani is a bounded model checker; ... arrays and other data structures are only explored to finite length—this is where the 'small scope hypothesis' comes in." [S74, P, SNIP]
- Tool-comparison page: reversing a cryptographic hash with a model checker is easy to try, "but good luck getting it to terminate!" [S72, P, SNIP]
- Newer Kani paper: "Harnesses without loop contracts provide bounded guarantees only". [S73, P, SNIP]
- The Kani Limitations page itself (FETCH) returned only a general statement and a list of topics; specific details were not in the retrieved text. [S71, P]

---

## 13. P language

**What it verifies.** Models of distributed designs as communicating state machines, with safety/liveness specs checked by systematic exploration; plus runtime/log conformance via PObserve.
- "P is a state-machine-based language for modeling and analysis of distributed systems." Teams across AWS (S3, EBS, DynamoDB, MemoryDB, Aurora, EC2, IoT) "have been using P to reason about the correctness of their system designs." [S75, P (CACM, AWS authors)]
- S3 strong read-after-write consistency: P "used to formally model and validate the protocol design"; "helped eliminate several design-level bugs early". [S75, P]
- PObserve (2023): "take structured logs from the execution of distributed systems and validate post hoc that they match behaviors allowed by the formal P specification"; described as "bridging the gap between the P specification of the system design and the production implementation (typically in languages like Rust or Java)". [S75, P]
- Origin: Windows 8 USB 3.0 driver stack modelled, implemented and model-checked in P. [S76, P, SNIP]
- Limitations: no explicit limitations page was fetched (gap). The sources describe what is covered (design models, and log conformance after the fact).

---

## 14. Stateright

**What it verifies.** Rust actor systems via an embedded explicit-state model checker; the same actors can be spawned on a real UDP network.
- "in contrast with model checkers such as TLC for TLA+, systems implemented using Stateright can also be run on a real network without being reimplemented in a different language." [S78, P]
- "Liveness checks via 'eventually' properties (experimental/incomplete)"; includes linearizability and sequential-consistency testers. [S78, P]
- Docs: "unlike many model checkers, Stateright is also able to spawn these actors on a real network." [S80, P]
- Example third-party use: Cykas protocol implementation checked with Stateright; "bounded verification of safety and liveness". [S81, S, SNIP (preprint)]
- Quint reference page: "Actors are plain Rust and can run on a real network, but the system must be built as Stateright actors." [S25b, S, SNIP]
- Stateright's own comparison with TLA+ page describes both abstract-model checking and implementation-level checking. [S79, P, SNIP]

---

## 15. Cross-tool: protocol case-study index

| Protocol | Tools/papers in these sources | What was covered | Notes from sources |
|---|---|---|---|
| TLS 1.3 | Tamarin (draft 10, draft 21) [S4, S5]; ProVerif/CryptoVerif (S&P 2017, cited via [S12b, S17]); F*/miTLS record layer [S56, S57]; EverParse formats [S58]; Fiat-Crypto in BoringSSL [S40] | Spec models; record layer implementation; wire format parsers; ECC arithmetic | Tamarin "analyse the specification only" [S4]. Handshake implementation proof incomplete in Everest [S56]. |
| QUIC | Ivy compositional testing [S31]; ProVerif/SPIN, ProVerif/Verifpal handshake models [S87, S88]; Tamarin QUICstep (privacy of a circumvention design) [S90, S]; Everest QUIC record layer [S56]; Dafny QUIC at CMU [S56] | Wire-protocol testing of implementations; handshake symbolic models; record layer proof | Ivy: 27 server errors; spec safety-only. [S31] |
| Signal | ProVerif, CryptoVerif, Tamarin, Verifpal [S12, S13, S85, S20] | X3DH/Double Ratchet symbolic and computational models | HAL 2026 paper: first automated analysis covering all Double Ratchet features; PCS proof ~40 min. [S85, P, SNIP] CryptoVerif Signal limited to one ratchet round [S86]. A hobby repo (positive-intentions) claims ProVerif + hax/F* for a Rust Signal; its own text admits it "doesn't prove implementation correctness" (tagged S, low reliability) [S91, S, SNIP]. |
| WireGuard | Tamarin [S15]; CryptoVerif [S14, S16]; SAPIC+ ProVerif+Tamarin [S84]; Gobra Go-code verification against Tamarin-derived I/O spec [S82]; EverCrypt Curve25519 in kernel [S56]; AutoTam [S83] | Protocol proofs; partial implementation proof (608 lines Go) | Arquint: DoS protection, metrics, load balancing removed; crypto and network stubs trusted. [S82] |
| 5G AKA | Tamarin (NDSS 2019), 5G handover, AKMA, 5G key-establishment stack [S6, S7, S3] | Spec drafts (TS 33.501) | Attack found in draft; no implementation analysed on Cremers' page. [S7] |
| Raft | Verdi-Raft (Coq) [S37, S38]; TLA+ with trace validation (etcd, CCF, MongoDB) [S27-S29]; Stateright [S78]; Isabelle student project [S53] | Full verified implementation (Verdi); spec + trace validation | Verdi shim bugs [S39]; MongoDB trace-check failure [S26, S27]. |
| Paxos | IronRSL (Dafny) [S62]; Aneris single-decree Paxos [S45b, S47]; Ivy/EPR Paxos variants and Isabelle Paxos [S36]; Apalache [S23] | Implementation refinement (IronRSL); protocol safety proofs | IronRSL: Paxos-based, 7,535 proof SLOC. [S62] |

---

## 16. Cross-tool: documented limitations by category (verbatim-leaning, no ranking)

- **Model vs code gap stated by the tool or its authors:** Verifpal ("does not prove that a deployed implementation is secure") [S19b]; Tamarin ("analyse the specification only") [S4]; CryptoVerif thesis ("no formal guarantee exists that the two interpretations of the specification match") [S16]; MongoDB ("The implementation must actually conform to the spec") [S26]; Ivy QUIC ("not an abstract implementation of the protocol") [S31].
- **Trusted/unverified boundary in verified-implementation projects:** IronFleet (spec, main loop, UDP interface, Dafny/.NET/Windows) [S62]; Verdi (shim, OCaml extraction/runtime) [S39]; F*/Everest (SMT solver, VC encoding, KaRaMeL, 26 F* soundness bugs) [S56]; Verus (spec, verifier, solvers, rustc) [S65]; Arquint (crypto/network stubs, setup code) [S82].
- **Termination/bounds:** Tamarin and ProVerif may not terminate [S1, S10]; Verifpal bounded [S19]; Apalache bounded [S24, S25]; Kani bounded by unwinding [S70]; Ivy tests run 100 events/1 connection [S31]; CryptoVerif polynomial sessions [S13].
- **Property classes not covered:** Ivy QUIC spec safety-only, no liveness/timing/congestion [S31]; Tamarin observational equivalence approximated [S1]; Arquint: equivalence/privacy at implementation level is "an open problem" [S82]; Kani: no concurrency [S72]; Kani/other: aliasing and data races not detected [S74].
- **Language/feature coverage:** Verus subset of Rust [S66]; Kani unsupported features [S71]; IronFleet limited to newly written code in Dafny [S62]; Gobra concurrency restrictions led to removal of metrics/load-balancing code from WireGuard case study [S82].
- **Effort/cost figures reported:** TLS Isabelle ~6 man-weeks [S49]; IronFleet ~3.7 person-years [S62]; MongoDB trace-check 10 weeks, abandoned [S27]; Realm OT test generation 4 weeks [S27]; Everest ~1.3M lines F*, years of effort, tooling pinned for years [S56]; Verus 31K proof lines for 6.1K code lines [S65]; Atmosphere 7.5:1 [S65b]; WireGuard Gobra case study: 608 Go lines + 3,936 lines annotations (32% generated) [S82].

---

## 17. Protocol-model-to-code frameworks found during the search

- **Arquint, Wolf, Lallemand, Sasse, Sprenger, Sefidgar, Basin, Müller - "Sound Verification of Security Protocols: From Design to Interoperable Implementations"** (FETCH of arXiv 2212.04171). [S82, P]
  - Tamarin MSR model -> per-role I/O specifications -> verified against Go (Gobra) or Java (VeriFast) code. Theorem 2: traces of the composed implementations are included in the Tamarin model, so "any trace property Φ that holds for R also holds" for the concrete system.
  - Assumptions: verifier soundness, trusted I/O and crypto contracts, injective message patterns, collision resistance, Dolev-Yao attacker only.
  - WireGuard: Tamarin ~250 lines MSR rules + 100 lines lemmas, ~3 min; 3 manual lemmas plus an oracle needed for termination; Go 608 verified lines; Gobra ~148 s (initiator), ~138 s (responder); interoperable with official WireGuard.
  - Not covered: setup code, crypto/network stubs, DoS protection (removed), metrics/load balancing (removed), non-DY attackers, equivalence properties.
- **AutoTam** (arXiv 2026, SNIP): language-first method; generates Tamarin models from protocol implementations in a DSL; WireGuard and signed DH examples; table of related work lists Arquint et al. and hax (Rust -> F*, ProVerif). [S83, P (preprint), SNIP]
- **Igloo** (Isabelle/HOL refinement -> VeriFast/Nagini code verification). [S52]
- **cv2fstar** (CryptoVerif -> F*). [S16]
- **Trillium/Aneris** (program refinement to TLA+ models in Coq/Iris). [S47]
- **DY*** (EuroS&P 2021): "mechanized symbolic verification framework that closes the gap between dependent types and dedicated provers like ProVerif and Tamarin" (F*). [S86, P, SNIP]
- **Refinement Methodology for Distributed Programs in Rust** (SMT-automated first-order models, 2025-26): compares itself with Iris-based work ("proofs in Rocq-based frameworks are not easily automatable and require extensive manual work") and IronFleet. [S89, P, SNIP]
- **MongoDB / etcd / CCF trace validation** (TLA+ with TLC). [S26-S29]
- **PObserve** (P specs against logs). [S75]

---

## 18. Source reliability notes

- Several results were low-quality aggregators or hobby sites (sota.io, positive-intentions.com, emergentmind, quant67.com, tractable.for-all.dev, various GitHub notes). They were not used for any claim beyond a flagged pointer (Signal row, S91). Claims on those sites such as "miTLS has been verified against the formal TLS 1.3 specification (RFC 8446)" were NOT used; they conflict with the Everest retrospective ([S56]) on whether the handshake implementation proof was completed. The conflict is recorded here and not resolved.
- Preprint arXiv IDs 2601-2611 were taken from search results as shown; only the snippets were read unless FETCH is marked.
- No source for ProVerif's TLS 1.3 paper (Bhargavan et al., S&P 2017), Signal's Cohn-Gordon et al., Noise Explorer, or a Dafny official limitations page was fetched directly; they are referenced only through third-party mentions.

---

## Source Register (all accessed 2026-11-07)

| ID | URL | Tag | Mode |
|---|---|---|---|
| S1 | https://tamarin-prover.com/manual/master/book/014_limitations.html | P | SNIP |
| S2 | https://tamarin-prover.com/manual/master/book/001_introduction.html | P | SNIP |
| S3 | https://tamarin-prover.com/index.html | P | SNIP |
| S4 | https://www.ndss-symposium.org/wp-content/uploads/2017/09/T3-tamarin.pdf | P | SNIP |
| S5 | https://tls13tamarin.github.io/TLS13Tamarin/docs/tls13tamarin-draft21.pdf | P | SNIP |
| S6 | https://arxiv.org/pdf/1806.10360 | P | SNIP |
| S7 | https://people.cispa.io/cas.cremers/tamarin/5G/ | P | FETCH |
| S8 | https://tamarin-prover.com/manual/master/book/010_modeling-issues.html | P | SNIP |
| S9 | https://bblanche.gitlabpages.inria.fr/proverif/ | P | SNIP |
| S9b | https://raw.githubusercontent.com/tamarin-prover/teaching/master/Tamarin-Tutorial-morning.pdf | P | SNIP |
| S10 | https://bblanche.gitlabpages.inria.fr/talks/Secret10.pdf | P | SNIP |
| S11 | https://drops.dagstuhl.de/storage/00lipics/lipics-vol380-lics2026/LIPIcs.LICS.2026.2/LIPIcs.LICS.2026.2.pdf | P | SNIP |
| S12 | https://radar.inria.fr/report/2019/prosecco/uid10.html | P | SNIP |
| S12b | http://www.link.springer.com/article/10.1186/s42400-026-00599-y | S | SNIP |
| S12c | https://lsv.ens-paris-saclay.fr/~baelde/1819/secu/proverif.pdf | S | SNIP |
| S13 | https://bblanche.gitlabpages.inria.fr/cryptoverif// | P | SNIP |
| S14 | https://bblanche.gitlabpages.inria.fr/cryptoverif/WireGuard/ | P | SNIP |
| S15 | https://www.wireguard.com/formal-verification/ | P | SNIP |
| S16 | https://theses.hal.science/tel-03933719/ | P | FETCH |
| S16b | https://inria.hal.science/hal-02396640v1/document | P | SNIP |
| S17 | https://bblanche.gitlabpages.inria.fr/talks/Cortier23.pdf | P | SNIP |
| S17b | https://inria.hal.science/hal-04271666v2/document | P | SNIP |
| S18 | https://verifpal.com/docs/comparison/ | P | SNIP |
| S19 | https://verifpal.com/docs/reference/ | P | SNIP |
| S19b | https://verifpal.com/docs/ | P | SNIP |
| S20 | https://verifpal.com/docs/introduction/ | P | SNIP |
| S21 | https://eprint.iacr.org/2026/1654 | P | SNIP |
| S21b | https://crates.io/crates/verifpal/1.6.4 | P | SNIP |
| S22 | https://link.springer.com/chapter/10.1007/978-3-030-65277-7_8 | P | SNIP |
| S23 | https://hal.science/hal-02280888/document | P | SNIP |
| S23b | https://link.springer.com/chapter/10.1007/978-3-031-30823-9_7 | P | SNIP |
| S23c | https://link.springer.com/chapter/10.1007/978-3-032-32519-8_8 | P | SNIP |
| S24 | https://radar.inria.fr/rapportsactivite/RA2019/veridis/uid40.html | P | SNIP |
| S25 | https://quint.sh/docs/model-checkers | S | SNIP |
| S25b | https://quint.sh/guides/formal-verification-tools | S | SNIP |
| S26 | https://www.mongodb.com/company/blog/engineering/conformance-checking-at-mongodb-testing-our-code-matches-our-tla-specs | P | FETCH |
| S27 | http://www.vldb.org/pvldb/vol13/p1346-davis.pdf | P | FETCH |
| S28 | https://github.com/etcd-io/raft/blob/main/tla/README.md | P | SNIP |
| S29 | https://github.com/tracevalidation/Raft | P | SNIP |
| S30 | https://ar5iv.labs.arxiv.org/html/2607.25333 | S | SNIP |
| S31 | https://dl.acm.org/doi/pdf/10.1145/3341302.3342087 | P | FETCH |
| S32 | https://github.com/microsoft/ivy/blob/master/doc/examples/quic/README.md | P | SNIP |
| S33 | https://pmc.ncbi.nlm.nih.gov/articles/PMC7363183/ | P | SNIP |
| S34 | https://www.galois.com/articles/quic-testing-a-quick-replication | S | SNIP |
| S35 | https://github.com/wilcoxjay/mypyvy | P | SNIP |
| S36 | https://ar5iv.arxiv.org/html/1710.07191 | P | SNIP |
| S37 | https://dl.acm.org/doi/10.1145/2813885.2737958 | P | SNIP |
| S38 | https://github.com/uwplse/verdi-raft/blob/master/coq-verdi-raft.opam | P | SNIP |
| S39 | https://cs.purdue.edu/homes/pfonseca/papers/eurosys2017-dsbugs.pdf | P | FETCH |
| S40 | https://adam.chlipala.net/papers/FiatCryptoSP19/ | P | SNIP |
| S41 | https://news.mit.edu/2019/fiat-cryptography-chrome-android-0617 | S | SNIP |
| S41b | https://www.csail.mit.edu/research/high-assurance-cryptography | P | SNIP |
| S42 | https://vst.cs.princeton.edu/ | P | SNIP |
| S43 | https://www.cs.princeton.edu/~appel/papers/verified-hmac-drbg.pdf | P | SNIP |
| S43b | https://www.cs.princeton.edu/~appel/papers/funspec_sub.pdf | P | SNIP |
| S44 | https://dl.acm.org/doi/10.5555/2831143.2831157 | P | SNIP |
| S45 | https://iris-project.org/aneris/ | P | SNIP |
| S45b | https://cs.au.dk/~timany/talks/files/2025_PLaID.pdf | P | SNIP |
| S46 | https://link.springer.com/chapter/10.1007/978-3-030-44914-8_13 | P | SNIP |
| S47 | https://iris-project.org/pdfs/2024-popl-trillium.pdf | P | SNIP |
| S47b | https://hal.science/hal-05647709/document | P | SNIP |
| S48 | http://link.springer.com/10.1007/978-3-319-89884-1_22 | P | SNIP |
| S49 | https://www.cl.cam.ac.uk/techreports/UCAM-CL-TR-440.html | P | SNIP |
| S50 | https://ar5iv.labs.arxiv.org/html/2105.06319 | P | SNIP |
| S51 | https://inria.hal.science/inria-00579015/document | P | SNIP |
| S52 | https://2020.splashcon.org/details/splash-2020-oopsla/28/Igloo-Soundly-Linking-Compositional-Refinement-and-Separation-Logic-for-Distributed- | P | SNIP |
| S53 | https://infsec.ethz.ch/education/studentProjects/isabelle.html | P | SNIP |
| S53b | https://www.research-collection.ethz.ch/server/api/core/bitstreams/bcc7d4a8-ec35-4db3-8eb5-9d92420b2500/content | P | SNIP |
| S54 | https://martin.kleppmann.com/2022/10/12/verifying-distributed-systems-isabelle.html | S | SNIP |
| S54b | https://pdfs.semanticscholar.org/f498/e6a3066dabbe2b811d189e8e1741e1145bce.pdf | S | SNIP |
| S55 | https://arxiv.org/pdf/2608.21935 | S | SNIP |
| S56 | https://dl.acm.org/doi/full/10.1145/3805702 | P | FETCH |
| S57 | https://www.microsoft.com/en-us/research/wp-content/uploads/2017/08/191.pdf | P | SNIP |
| S58 | https://www.microsoft.com/en-us/research/publication/everparse/ | P | SNIP |
| S59 | https://www.microsoft.com/en-us/research/blog/project-everest-reaching-greater-heights-in-internet-communication-security/ | P | SNIP |
| S60 | https://6826.csail.mit.edu/2020/lec/l04-everest.pdf | P | SNIP |
| S62 | https://www.microsoft.com/en-us/research/wp-content/uploads/2015/10/ironfleet.pdf | P | FETCH |
| S63 | https://www.microsoft.com/en-us/research/wp-content/uploads/2017/06/ironfleet-cacm.pdf | P | SNIP |
| S64 | https://github.com/microsoft/Ironclad/blob/main/ironfleet/README.md | P | SNIP |
| S65 | https://verus-lang.github.io/paper-sosp24-artifact/assets/paper-20240921-162720-b7db935.pdf | P | SNIP |
| S65b | https://dl.acm.org/doi/10.1145/3625275.3625401 | P | SNIP |
| S66 | https://verus-lang.github.io/verus/guide/ | P | SNIP |
| S67 | https://arxiv.org/html/2303.05491 | P | SNIP |
| S68 | https://iris-project.org/pdfs/2026-oopsla-refinedrust2.pdf | P | SNIP |
| S68b | https://github.com/kosumic/Verustd | S | SNIP |
| S69 | https://dl.acm.org/doi/10.1145/3808325 | P | SNIP |
| S70 | https://model-checking.github.io/kani/soundness.html | P | FETCH |
| S71 | https://model-checking.github.io/kani/limitations.html | P | FETCH |
| S72 | https://model-checking.github.io/kani/tool-comparison.html | P | SNIP |
| S73 | https://arxiv.org/html/2607.01504 | P | SNIP |
| S74 | https://arxiv.org/html/2510.01072v3 | P | SNIP |
| S75 | https://cacm.acm.org/practice/systems-correctness-practices-at-amazon-web-services/ | P | SNIP |
| S76 | https://p-org.github.io/P/casestudies/ | P | SNIP |
| S77 | https://p-org.github.io/P/ | P | SNIP |
| S78 | https://github.com/stateright/stateright | P | SNIP |
| S79 | https://www.stateright.rs/comparison-with-tlaplus.html | P | SNIP |
| S80 | https://docs.rs/stateright/latest/stateright/ | P | SNIP |
| S81 | https://arxiv.org/html/2603.14690v1 | S | SNIP |
| S82 | https://arxiv.org/pdf/2212.04171 | P | FETCH |
| S83 | https://arxiv.org/html/2606.19937v1 | P | SNIP |
| S84 | https://www.ndss-symposium.org/wp-content/uploads/2024-364-paper.pdf | P | SNIP |
| S85 | https://hal.science/hal-05589499/document | P | SNIP |
| S86 | https://inria.hal.science/hal-03178425/file/dy-star-eurosp21.pdf | P | SNIP |
| S87 | https://onlinelibrary.wiley.com/doi/10.1155/2021/1630223 | P | SNIP |
| S88 | https://scispace.com/pdf/formal-analysis-of-quic-handshake-protocol-using-symbolic-3numg0riuw.pdf | P | SNIP |
| S89 | https://dl.acm.org/doi/10.1145/3763119 | P | SNIP |
| S90 | https://www.sos-vo.org/group/hcss_conference/2026/krahenbuhl | S | SNIP |
| S91 | https://positive-intentions.com/docs/research/Security%20audit/signal-protocol-security-audit/formal-verification/ | S (low reliability) | SNIP |

Distinct sources: 108 register entries (11 read in full via fetch, the remainder as search excerpts).

---

## Open Questions

1. Which specific TLS 1.3 handshake properties did ProVerif and CryptoVerif establish (Bhargavan et al. S&P 2017), and where does that paper itself state the model-to-implementation boundary? Not fetched directly.
2. Do Cohn-Gordon et al. (Signal, EuroS&P 2017) and Kobeissi et al. (Noise Explorer) state any link from the models to Signal or Noise implementations? Only third-party mentions retrieved.
3. For the Verifpal 2020 paper versus the 2026 documentation: the 2020 abstract states unbounded sessions and Coq formalization, while 2026 pages describe bounded sessions. Which statement applies to which engine version?
4. What exactly does Tamarin's SAPIC+ pipeline guarantee about equivalence between process-calculus input and the generated multiset-rewrite model, and are there documented translation gaps?
5. Is there a documented limitations section for P (language and checker backends), PObserve conformance scope, and Stateright (explicit-state limits, symmetry reduction)? None was fetched.
6. How many of the TLA+ spec-to-code conformance attempts (beyond MongoDB, etcd raft, CCF) report failures or abandonment? Only MongoDB's failure is documented here in detail.
7. What did Dafny's own documentation state about trusted components (compiler, axioms, `{:axiom}`), and how does Dafny 3.4.0 used for IronFleet relate to current Dafny?
8. For Verus: are there published case studies of network-protocol implementations (not storage/OS) verified end to end, and for Verus-based distributed system work (IronKV-line, Anvil), what are the stated TCB and fault-model assumptions?
9. What precisely did Verdi's Coq-to-OCaml extraction trust, according to Verdi's own papers (S37 only read as a snippet; the Chinese-language blog claim about unverified parts was not used)?
10. Which Kani deployments exist on real network protocol code (for example s2n-quic, Firecracker), and what bounds did they use? Not searched.
11. For Isabelle: is there any Isabelle/HOL work that targets a deployed protocol implementation (rather than the Paulson-style model), apart from Igloo and its router case study? Not found in this pass.
12. What are the ProVerif-to-Rust/F* extraction paths (hax, hacspec) and their documented assumptions? Only appear in third-party and preprint tables here.
13. The conflicting statements about miTLS verification status (Everest retrospective versus secondary web pages): is there a primary release note that states which handshake modules carry proofs?
14. Which of the effort figures (person-years, lines of proof, weeks) are comparable across tools, given different scopes? The sources do not report on a common basis.
15. For Rocq/Coq Raft (Verdi-Raft), the SoK and other 2025-26 sources mention end-to-end extracted-implementation guarantees; which of these state the exact runtime-shim assumptions after the 2017 bug study?
