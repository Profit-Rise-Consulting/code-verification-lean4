# 04 - Other tools: how they verify protocols and their implementations (comparators)

**Phase 1b file (rewritten 2026-11-07).** Counts: **[FULL] entries: 83 | [CODE] entries: 0 | Unverified leads: 16** (see the final section; leads are excluded from evidence). No Lean repository is relied on in this file, so no [CODE] inspection applies. No verdict on the hypothesis is stated anywhere in this file.

## How to read this file

- Access date for every entry: **2026-11-07**. Tags: **[P]** primary (tool docs, paper, project repo, author page), **[S]** secondary. Depth: **[FULL]** = the primary page, paper or README was fetched as full text and the passages relevant to scope, numbers and limitations were read. For long PDFs (for example Trillium, Grove, Verus, the Dafny reference manual, Paulson, Signal, Everest) the abstract, the evaluation/limitations/assumptions sections and keyword-located passages were read; the remaining pages were not read line by line. Where a quote is given it was checked against the fetched text.
- Sub-claim tags (from CONVENTIONS.md): H1 expressiveness, H2 provability at scale, H3 spec-to-implementation link, H4 generality ("any codebase"), H5 inference ("therefore"), D definitions of verify/check. They mark what the entry bears on; they are not findings.
- Separate dossiers (other agents, `research/references/`) cover Everest, IronFleet/Verus, Verdi, Fiat-Crypto, s2n-tls SAW, Tamarin/ProVerif/CryptoVerif case studies, Ivy QUIC and TLA+ trace validation. This file records the comparator-level facts and points there for depth; it does not duplicate them.
- Search-engine snippets were used only to find sources. Aggregator and hobby sites were not used as evidence (see section 18).
- Inline cites `[S#]` resolve to the Source Register at the end.

---

## 0. One-table overview (what each tool's input is checked against)

"Target" is the artifact the tool's result is about, as stated in the sources.

| Tool | Target of the check | Stated implementation link | Bound on scope | Evidence |
|---|---|---|---|---|
| Tamarin | Protocol model (multiset rewriting), symbolic Dolev-Yao | None in the tool. Research frameworks derive I/O specs for Go/Java code (Arquint et al.) or generate models from a DSL (AutoTam) | Unbounded sessions; may not terminate | S1, S2, S5, S6, S52, S76 |
| ProVerif | Protocol model (applied pi / Horn clauses), symbolic | None in the tool. DY* (F*) is a separate effort | Unbounded sessions; may not terminate; false attacks possible | S8-S10, S79 |
| CryptoVerif | Protocol model, computational (game sequences) | Translations to OCaml and to F* are listed on the tool page; thesis builds cv2fstar | Polynomial number of sessions | S11, S12, S14 |
| Verifpal | Protocol model, symbolic, fixed 25 primitives | None; docs say a pass does not show an implementation is secure | Bounded sessions and search depth | S17-S21 |
| TLA+ / TLC / Apalache | Specification (state machine) | Add-on techniques: trace validation, test generation (MongoDB, etcd raft) | TLC finite instances; Apalache bounded length or inductive invariant | S22-S28 |
| Ivy / mypyvy | Protocol models; Ivy also spec-based testing of real implementations | Ivy QUIC: executable monitor + randomized tester (testing, not proof) | Decidable-fragment proofs; tests run finite executions | S29-S34 |
| Coq: Verdi | Implementation written in Coq, extracted to OCaml | Extraction; unverified shim | Proof | S35, S36 |
| Coq: VST | C programs (Clight) vs functional specs | Proof about C code | Proof | S37-S39 |
| Coq: Fiat-Crypto | Generator of C arithmetic code | Proof about generator output | Proof | S40, S41 |
| Coq + Iris: Aneris / Trillium / Grove | Distributed programs in AnerisLang / GooseLang | Trillium: refinement to TLA+-style models; Grove: Go code | Proof | S42-S46 |
| Isabelle/HOL | Protocol models (inductive method); refinement (Igloo) | Igloo: links Isabelle models to code verified in VeriFast/Nagini/Gobra | Proof | S47-S51 |
| F* / Everest | Implementation code (Low*) extracted to C | Code is the artifact; TLS 1.3 handshake implementation proof not completed | Proof | S53, S54 |
| Dafny / IronFleet | Implementation code, three refinement layers | All layers in Dafny, compiled to C# | Proof | S55-S57, S62 |
| Verus | Rust implementation code | Code is the artifact | Proof (SMT) | S58-S61 |
| Kani | Rust implementation harnesses | Code is the artifact | Bounded (unwinding) unless loop contracts | S63-S68 |
| P | Models as communicating state machines; PObserve checks logs against P specs | PObserve: post-hoc log validation against P specs | Systematic exploration | S69-S71 |
| Stateright | Rust actor models; same actors can run on UDP | Same code is model checked and run | Finite state spaces | S72-S75 |

---

## 1. Tamarin

**What it verifies.** Protocol models (multiset rewrite rules) in the symbolic model.
- "If the tool's automated proof search terminates, it returns either a proof of correctness (for an unbounded number of role instances and fresh values) or a counterexample, representing an attack that violates the stated property." [S2]
- Limitations page: "Tamarin operates in the symbolic model and thus can only capture attacks within that model, and given a certain equational theory. Currently, apart from the builtins, only subterm-convergent theories are supported. The underlying verification problems are undecidable in general, so Tamarin is not guaranteed to terminate." [S1]
- "the observational equivalence mode currently only (soundly) approximates observational equivalence by requiring a strict one-to-one mapping between rules". [S1]
- Modeling-issues page: non-subterm-convergent equations "can result in infinite loops or incorrect results". [S3]
- NDSS 2017 tutorial slides on TLS 1.3: "We analyse the specification only"; "We assume black-box cryptography". [S5]

**Case studies and effort figures.**
- 5G AKA (CCS 2018, Basin et al.): model ~500 lines, 124 lemmas (~1,000 lines), Tamarin ~5 hours; "Tamarin's fully automatic mode fails to prove relevant security properties and even extremely weak properties such as the full executability" without the authors' proof strategies; manual proofs "not feasible due to the size of the search space". [S6] Cremers' project page: vulnerability found in the Feb 2018 draft of TS 33.501; the page does not claim an implementation was analysed. [S7]
- TLS 1.3 (CCS 2017, Cremers et al.), as reported by the Verifpal documentation (a competitor's page, [S]-grade for this claim): about three months of modeling work, over 10 GB RAM to load, up to 100 GB to prove. [S17]
- WireGuard: Tamarin symbolic proof (Donenfeld/Milner) with correctness, key agreement, KCI resistance, UKS resistance, key secrecy, forward secrecy, session uniqueness, identity hiding; the project page calls the accompanying paper "a draft work in progress". [S13]
- Implementation link outside the tool: Arquint et al. (section 17) and AutoTam [S76].

---

## 2. ProVerif

**What it verifies.** Symbolic models in the applied pi calculus; reachability and equivalence properties via Horn clauses. [S8]
- Blanchet's 2010 slides: "Limitations: Risk of non-termination. Risk of false attacks."; the abstraction ignores repetition ("repetitions of actions are ignored: the clauses can be applied any number of times") and can cause "(rare) false attacks". [S9]
- Delaune's invited paper (LICS 2026, "25 Years of ProVerif") concludes with "ongoing challenges, including usability and interoperability between verification approaches". [S10]
- Verifpal's comparison page (competitor's description): ProVerif "has no general termination guarantee, though one holds for the tagged-protocol class"; `cannot be proved` is "inconclusive". [S17]
- TLS 1.3 (Bhargavan, Blanchet, Kobeissi, S&P 2017) as reported by [S17]: 1,030 lines of ProVerif, ~70 minutes for the proof suite, ~3 person-weeks for an expert; the primary paper itself could not be fetched (see leads).
- Signal Double Ratchet (HAL 2026, Cheval/Jacomme/Richards): first formal analysis covering all Double Ratchet features including out-of-order arrival, in ProVerif; PCS proof took 40 min / 2.7 GB; "Our analysis uncovered three attacks on the protocol, two of which we confirmed to be present in the main implementation, and a third which exists in the specification." [S78]
- Noise Explorer: validates a Noise handshake pattern then translates it "into a model ready for automated verification and also into a production-ready software implementation written in Go or in Rust"; analyses more than 57 patterns. [S16]
- WireGuard unified analysis (NDSS 2024) in SAPIC+ with ProVerif and Tamarin: agreement, secrecy and PFS ~15 minutes each on a 256-core server; anonymity ~9 hours (guv-based fix) or ~2 hours (psk-based fix). [S77]

---

## 3. CryptoVerif

- Tool page: "CryptoVerif is an automatic protocol prover sound in the computational model... The generated proofs are proofs by sequences of games... These proofs are valid for a number of sessions polynomial in the security parameter, in the presence of an active adversary." It also "provides translations of protocol models to OCaml for generating implementations and to F* for generating implementations and reasoning about them". "This software is under development; please use it at your own risk." [S11]
- WireGuard (EuroS&P 2019): model files and scripts for CryptoVerif 2.01pl1; the WireGuard site describes the Lipp thesis as "a mechanised cryptographic proof of the entire WireGuard protocol, including transport data messages, in an ACCE-like computational model", proving correctness, message secrecy, forward secrecy, mutual authentication, KCI and UKS resistance, replay resistance of the first message. [S12, S13]
- Lipp thesis (FETCH, Phase 1): "there is frequently a gap between a cryptographic security analysis and an implementation that have both been based on a protocol specification: no formal guarantee exists that the two interpretations of the specification match". Contribution: `cv2fstar`, "a compiler from CryptoVerif models to executable F* specifications using the HACL* verified cryptographic library as backend", demonstrated on Needham-Schroeder-Lowe. [S14]
- Signal (Cohn-Gordon et al., computational pen-and-paper proof, full text): "We extract from the implementation a formal description of the abstract protocol, and define a security model which can capture the 'ratcheting' key update structure... We then prove the security of Signal's key exchange core in our model... We have found no major flaws in the design". Its Limitations section: "As a first analysis of a complex protocol, we have chosen (some) simplicity over a full analysis of all of Signal's features." Its related-work text describes Kobeissi, Bhargavan and Blanchet using "ProVerif and CryptoVerif to analyze a simplified variant of Signal specified in a JavaScript variant called ProScript", a variant "that, e.g., does not use symmetric ratcheting", and automatically extracting ProVerif models from the ProScript code "that consider a finite number of se[ssions]" (text cut at that point in the fetched copy). [S15]
- DY* paper (EuroS&P 2021) on the earlier Signal analysis by Kobeissi et al.: "The ProVerif analysis of the Signal protocol was limited to two messages (three ratcheting rounds), at which point the analysis already took 29 hours. (With CryptoVerif, the analysis of Signal has to be limited to just one ratcheting round.)" DY* itself is a framework in F* for "cryptographic protocol code" that "accounts for advanced protocol features like unbounded loops and mutable recursive data structures, as well as low-level implementation details like protocol state machines and message formats", demonstrated on a Signal analysis "based on an interoperable implementation". [S79]

---

## 4. Verifpal

**What it verifies.** Protocol models in its own DSL; symbolic active attacker; fixed 25 primitives; five query forms (confidentiality, authentication, freshness, equality, linkability). [S17]
- Comparison page: "Verifpal searches within explicit limits; ProVerif and Tamarin can reason about unbounded sessions." "Finishing the search does not prove security." [S17]
- Result codes: `PASS [search exhausted at 2 sessions]` means the search found no contradiction at that session count and "can still miss attacks at two sessions because it does not try every possible construction or interaction"; `PASS [search truncated: term depth]` means a depth limit excluded at least one candidate; `FAIL` "is not a machine-checkable certificate, and it describes the model rather than an implementation". [S17]
- Default: "two sessions per principal". Attack search is "bounded and heuristic; no completeness claim". [S17]
- Quickstart/reference: it "does not prove that a deployed implementation is secure." [S19] Introduction: model rules omit byte lengths, timing, implementation bugs and computational probabilities. [S20]
- On connecting model to code, the same page says transferring a model result to code requires "showing that every implementation execution is represented and that satisfying the modeled property implies the intended implementation property" (paper §5.1) and gives the encoding-ambiguity example (RFC 5116 §3.3: a free pairing constructor excludes `(ab,c)` vs `(a,bc)` ambiguity). [S17]
- The page reports soundness-audit findings from the Verifpal authors' paper (§4): gaps between ProVerif translation stages and published proofs, and reproduced Tamarin 1.12.0 reports (variable collision, channel alias) yielding false `verified` results. It states the audit "does not refute either core method". Note: authored by the Verifpal developer. [S17]
- It also states that independent checking exists in other tools: "scyther-proof produces Isabelle proofs, and DyLean uses Lean." [S17]
- 2026 IACR ePrint 2026/1654 ("From Toy to Instrument: Seven Years of Verifpal") describes a replacement analysis engine and characterizes the tool as a bounded counterexample finder. [S21]

---

## 5. TLA+ / TLC / Apalache

**What it verifies.** A specification (state machine plus temporal properties), not code. Connection to code is a separate step.
- TLC (Quint documentation): "an explicit-state model checker" that enumerates states; the state space must be small enough. [S23] Apalache: a "symbolic bounded model checker"; "cannot check an arbitrary number of steps"; the number of steps must be chosen (default `--max-steps` 10 in that documentation). [S23]
- Apalache paper (Konnov, Kukovec, Tran, OOPSLA 2019, "TLA+ Model Checking Made Symbolic"): the tool checks TLA+ via SMT, with bounded executions and inductive-invariant checking for fixed parameters; read abstract, evaluation and conclusions of the fetched text. It states its assumptions as fixed finite parameters and finite data structures, and checks "only finite executions up to a given bound". Evaluation compared Apalache with TLC on Paxos-, Raft- and Byzantine-style specifications. [S22]
- Quint's tool reference lists Apalache and TLC and says the gap between a specification and a running implementation has to be "closed separately, with conformance testing techniques". [S24]

**Spec-to-implementation bridges (documented experience; deep treatment is in the TLA+ trace-validation dossier).**
- MongoDB, VLDB 2020 eXtreme Modelling in Practice (Davis et al.) [S28]. Trace-checking server vs `RaftMongo.tla` (345 lines): "MBTC was impractical for our system". Prototype 10 weeks, 2 engineers (event tracing 570 lines C++, 3 weeks of spec updates to 252 TLA+ lines, post-processor 484 Python). Only 5 handwritten tests and 1 randomized test could be trace-checked; 1 handwritten test passed, 4 violated the spec. Spec/impl differences: arbiters not modelled, initial-sync nodes counted toward quorum, two leaders briefly possible, term propagation, oplog copying. State space grew from 42,034 to 371,368 states; check time from 2 s to 14 min.
- Same paper, test-case generation for Realm Sync OT: 6 array operations, 21 merge rules, ~1,000 lines C++; TLA+ spec 795 lines; 4,913 generated tests; branch coverage 100% vs 21% (36 handwritten tests) and 92% (~8M fuzz executions); found a non-terminating ArraySwap/ArrayMove merge (StackOverflowError) in the C++. Authors: worked because the code was "a self-contained function with defined inputs and outputs" and the state space was manageable. [S28]
- MongoDB 2025 engineering blog: "The traces never matched our specification"; "10 weeks"; "It's hard to snapshot a multithreaded program's state"; "The implementation must actually conform to the spec." [S27]
- etcd raft: TLA+ specification of the library's core algorithm "serves dual purposes: it not only helps to verify the correctness of the model but also facilitates model-based trace validation"; spec checked with TLC; implementation adherence "is fortified through trace validation"; trace points are injected into the Go code under a build tag; `Traceetcdraft.tla` consumes NDJSON traces. [S25] The `tracevalidation/Raft` repository is the CCF-style Raft trace-validation companion (1 star, no description; research-grade). [S26]

**Documented limitations.** TLC explicit-state explosion; Apalache bounded length; separate conformance step with reported cost and failure cases [S22-S24, S27, S28].

---

## 6. Ivy and mypyvy

**What they verify.**
- Ivy (CAV 2020 tool paper): "Compared to proof assistants, Ivy provides domain-specific support for decidable proof automation... On the other hand, Ivy relies on a vastly larger trusted computing base than typical proof assistants. Moreover, Ivy has no mechanism of reflection, and thus cannot be used for meta-reasoning about programs and program transformations." [S31]
- mypyvy (README): "takes an input file describing a symbolic transition system and can perform various tasks such as inductive invariant checking and inference." Input is a transition system, not code. [S33]
- Paxos Made EPR (Padon, Losa, Sagiv, Shoham, PLDI 2017 line): safety of Paxos, Multi-Paxos, Vertical, Fast, Flexible and Stoppable Paxos in a decidable fragment; "We decided to verify the designs of the protocols and not their implementations" (as read in the full text; the paper discusses abstraction of the protocols); Disk Paxos in Isabelle/HOL cited at ~6,500 lines of proof script. Safety only; the full text discusses liveness as outside the approach. [S34]

**QUIC case study (McMillan and Zuck, SIGCOMM 2019; full text).** [S30]
- Method: a formal Ivy wire specification of QUIC used as an executable monitor and SMT-based randomized tester that takes one role: "our specification is not an abstract implementation of the protocol, but is instead an executable monitor".
- Result: four implementations tested; "27 errors in the server implementations" (13 protocol errors, 12 crashes, 2 failures to make progress); 4 potentially exploitable (includes a Heartbleed-like memory leak); 4 errors attributed partly to standard ambiguities.
- Scope limits: spec omits 0-RTT, version negotiation, Retry, RETIRE_CONNECTION_ID; "captures only safety properties"; "We do not cover any statement in the QUIC document on recovery and congestion control"; encryption/decryption, OS and network are black boxes; TLS handshake events use a real TLS implementation; "a large portion (perhaps most) of the statements in the standard were not usefully testable". Runs limited to 100 protocol events and one connection; generation ~10 Hz; ~4 weeks of engineer time. Presented as testing, not proof.
- README of the Ivy QUIC specification: "The specification as it stands has a number of limitations. First, it covers only a subset of features of QUIC... Second, the specification does not deal with quantitative time, meaning certain aspects of the QUIC specification relating to transmission rates and timeouts can't be stated." [S29]
- Galois blog (secondary, vendor): reports the same project as a spec-based testing approach. [S32] Deep treatment: Ivy QUIC dossier.

---

## 7. Coq / Rocq

### 7.1 Verdi, Verdi-Raft (dossier: Verdi)
- Verdi-Raft opam metadata: "verified implementation of the Raft distributed consensus protocol in Coq, constructed using the Verdi framework". [S36]
- EuroSys 2017 empirical bug study (Fonseca et al.; full text): 16 bugs across IronFleet, Verdi, Chapar after >8 months of searching; 11 in shim layers, 2 spec/client-assertion, 3 in verification tooling; "No protocol bugs were found in the verified systems"; shim components "typically only a few hundred lines of source code"; 9 of 11 shim bugs crashed or hung servers. Verdi had 8 shim-layer bugs (partial TCP reads, unescaped marshaling, non-atomic log write, snapshot truncation, transient open error, oversized UDP packets, stack exhaustion in generated OCaml). The unverified comparison sample had 12 protocol bugs. [S35]

### 7.2 VST (Verified Software Toolchain)
- Site: VST assures "with machine-checked proofs that the assertions claimed at the top of the toolchain really hold in the machine-language program"; implemented in Coq; the user supplies function specifications and loop invariants. [S37]
- Beringer, Petcher, Ye, Appel (USENIX Security 2015): proofs in Coq that the OpenSSL HMAC-SHA256 implementation "correctly implements its FIPS functional specification" and that the spec yields the expected cryptographic properties; combines VST, FCF and CompCert. [S38]
- Appel et al., mbedTLS HMAC-DRBG: crypto security of the functional spec (hybrid game proof) and correctness of the C implementation, "assuming that the DRBG's initial state is ideally random". [S39]
- Targets: SHA-256, HMAC, HMAC-DRBG primitives and their C code; not a protocol state machine. Side-channel properties are outside these proofs (stated scope of the papers' security model; entries read for scope, not for a full list of exclusions).

### 7.3 Fiat Cryptography (dossier: Fiat-Crypto)
- SP 2019: "first verified high-performance implementation of P-256"; implementations from the library "were included in BoringSSL ... for Chrome, Android, and CloudFlare". [S40] MIT News: "about 90 percent of secure Chrome communications currently run their code" [S41, S].
- It verifies a code generator for arithmetic, not a TLS state machine. [S40]

### 7.4 Iris, Aneris, Trillium, Grove
- Aneris site: "higher-order distributed concurrent separation logic" mechanized in Coq/Rocq. [S42] Timany's 2025 PLaID talk lists systems verified with Aneris: load balancer, causally consistent KV store, two-phase commit, CRDTs, reliable communication over a UDP-like network with session types, single-decree Paxos, snapshot-isolation database. [S45]
- Trillium (POPL 2024): shows "that two distributed protocols, two-phase commit and single-decree Paxos, refine their abstract TLA+ specifications" and calls it "the first foundationally verified proof that a concrete implementation of a distributed protocol correctly implements an abstract TLA+ specification". Describing IronFleet it says it "does not support node-local concurrency" and uses "a pen-and-paper argument for proving liveness". [S43]
- Fairneris (liveness extension, preprint): "first program logic for verification of both safety and liveness properties of higher-order and concurrent implementations of distributed systems, under a flexible network fairness assumption"; case study is a higher-order RPC. [S46]
- Grove (SOSP 2023): verifies Go components including vKV (multi-threaded KV store with reconfiguration, primary/backup replication, crash recovery, leases); 2,605 lines of Go total, 2,435 verified with 28,077 lines of Coq spec and proof, network and filesystem libraries marked Trusted; "12x the lines of proof as lines of code"; 1,597 lines of Coq for Grove's new reasoning principles; "One limitation is that Grove is only able to verify safety properties... Grove cannot verify liveness properties". vKV reaches 67-73% of Redis on a single core. [S44]

---

## 8. Isabelle/HOL

**Protocol-level.**
- Paulson, inductive method (JCS 1998): protocols as inductively defined sets of traces, Isabelle/HOL; the text states there is "no restriction to finite-state systems" (read in the full text). [S48] TLS case study (TR 440): "six man-weeks of effort and three minutes of processor time" were needed for the TLS (SSL 3.0 descendant) proofs. [S47]
- Kleppmann (blog, 2022): teaching-level example of modeling a distributed algorithm and proving it in Isabelle/HOL; secondary-grade for the claims. [S49]

**Refinement toward code.**
- Igloo (OOPSLA 2020, conference page with abstract): "We formalized our framework, Igloo, in Isabelle/HOL. We report on three case studies, a leader election protocol, a replication protocol, and a security protocol, for which we refine formal requirements into program specifications that we implement in Java and Python and prove correct using the VeriFast and Nagini tools." [S50]
- Trillium authors' comparison: "Igloo proves a particular kind of extensional safety properties" and refines each node's STS annotated with IO operations. [S43]
- "Formal Verification of a Next-Generation Internet Router" (ETH Zurich; SCION border router), full text. Abstract: protocol models "by refinement in Isabelle/HOL" and "an automated program verifier to prove that the router's Go code satisfies memory safety, crash freedom, freedom from data races, and adheres to the protocol model. Both verification efforts are soundly linked together." The protocol models start "under a weak adversary" and are refined "under a strong symbolic (Dolev-Yao) adversary"; Gobra checks the Go code against an automatically generated I/O specification. Figures: 13,400 lines of specifications and annotations (900 for the I/O specification), another 2,400 lines of trusted specifications for the Go standard library and third-party libraries, "2.8 lines of annotation per line of code", "roughly 2.5 person years", Gobra run time "three hours on a commodity laptop"; 12 functions verified except some error-handling paths because of two implementation errors found and reported; three more verified partially because of Gobra performance. Found "five attacks, including a critical attack that allowed a malicious sender to fabricate arbitrary forwarding paths" and bugs in the implementation. [S51]

---

## 9. F* and Project Everest (miTLS, HACL*, EverParse) - dossier: Everest

All Everest figures below come from the 2026 TOPLAS retrospective (full text) unless noted. [S53]

**What it verifies.** Implementation code written in F*/Low*, extracted to C/Wasm/Rust by KaRaMeL; SMT (Z3) automation.
- Verified: TLS-1.2 and 1.3 record layers (with a cryptographic security reduction), QUIC record layer, HACL* crypto (primarily C), EverParse parsers/serializers (non-malleable), including TLS wire formats (293 datatypes in the 2019 paper). [S53]
- Record layer paper (Delignat-Lavaud et al., S&P 2017): "We build and verify a reference implementation of the TLS record layer and its cryptographic algorithms in F*... We reduce the high-level security of the record layer to cryptographic assumptions on its ciphers. Each step in the reduction is verified by typing an F* module." [S54]
- Not completed: "we did not complete verifying our implementation of the TLS handshake"; the pen-and-paper handshake/key-schedule proof (state-separating proofs) was "only checked manually" and "we did not manage to translate this pen-and-paper proof to the verified code of miTLS-fstar". Rest of QUIC (CMU, Dafny) "was only proven safe, rather than functionally correct or cryptographically secure." [S53]

**Deployment.** NSS (~17 HACL* algorithms), WireGuard in Linux (EverCrypt Curve25519), Python hash/HMAC, Tezos, ElectionGuard; EverParse in Windows Network Virtualization (>100 message kinds across four protocols, ~5,000 lines of 3D spec to ~30,000 lines C, 5 years in production); miTLS handshake used as default for Microsoft msquic during development, later superseded by Schannel. [S53]

**Documented limitations / TCB.**
- F* "lacks a kernel checker"; trusts VC encoding and SMT solver; 26 soundness bugs found in F* (3 open); KaRaMeL is unverified OCaml and 6 bugs escaped formalization, internal checks and the C compiler. [S53]
- Scale: ~1.3M lines F* source, ~622,000 proof obligations, ~4,200 files; proof-to-code about 2:1 in executable-focused projects; HACL* 212,000 lines F* to 101,000 lines C; a doubly linked list library took one member >6 months. [S53]
- Maintenance: HACL* pinned to old F* in 2017-18; re-upgrade took nearly a year; Z3 pinned to 4.8.5 (2019) until 2024. [S53]
- Hindsight: initially judged too costly to relate executable code to an idealized model before proving crypto security; "might have separated concerns" (paraphrase). [S53]

---

## 10. Dafny and IronFleet - dossier: IronFleet/Verus

**Dafny.** Dafny reference manual (full text fetched, 534K characters; read the sections on `assume`, `{:axiom}`, `{:extern}`, `{:verify false}` and compilation):
- `assume` "lets the user specify a logical proposition that Dafny may assume to be true without proof. If in fact the proposition is not true this may lead to invalid conclusions." [S62]
- On body-less methods/functions: "By supplying an `{:extern}` or `{:axiom}` attribute, you are taking that responsibility into your own hands." [S62]
- Compilation targets listed in the manual include C#, Java, JavaScript, Go, Python (and C++); the manual has per-target feature-support tables. Correctness of the compiler is not part of what the verifier proves. [S62] (The Dafny manual makes no protocol-specific claim.)

**IronFleet (SOSP 2015; full text).** [S55]
- "the first methodology for automated machine-checked verification of the safety and liveness of non-trivial distributed system implementations"; three layers: high-level spec, distributed-protocol layer, implementation layer, with refinement between them.
- Table (SLOC): IronRSL spec 6 / impl 2,941 / proof 7,535 / verify time 152 min; IronKV 6 / 1,340 / 2,937 / 42 min; high-level specs 85 and 34. Implementation proof-to-code ratio "3.6 to 1". Effort: "approximately 3.7 person-years".
- CACM 2017 version (full text) reports different totals: "Our ratio of proof annotation to implementation is 7.7:1 (5.4:1 if liveness proof annotations are excluded)"; Table 1 totals: spec 1,400, impl 5,114, proof 39,253, 395 minutes; IronRSL distributed-protocol proof 12,450, IronKV 6,817, TLA library 1,824; the same 3.7 person-years. Performance "competitive with unverified reference implementations" (IronRSL 18,200 req/s, IronKV 28,800 req/s). [S56]
- TCB: "The spec for each system is trusted, as is the brief main-event loop"; trusted UDP spec (`Init`, `Send`, `Receive`); network may drop, delay or duplicate but not tamper; correctness of Dafny, .NET, Windows, hardware assumed. [S55]
- Liveness assumptions for IronRSL: quorum, scheduler frequency, bounded delay, burst, clock error, no overflow. [S55]
- Limits stated: crash recovery not verified; reduction-obligation proof and memory-exhaustion proof "future work"; newly written code in a verification-friendly language; no Byzantine guarantee; performance below optimized unverified baselines; "more up-front development effort". [S55, S56]
- The released Ironclad repository README (experimental) lists Dafny v3.4.0 and .NET 6.0 for IronFleet. [S57]
- Empirical bug study (Fonseca et al.): IronFleet had 1 specification bug (spec did not require exactly-once semantics; a 7-line patch disabling deduplication still verified) and 3 verification-tool bugs (NuBuild mis-parsing prover crashes, SIGINT handling in Boogie, 32/64-bit z3 mismatch); no protocol bugs. [S35]
- Later work contrasts: IronFleet's refinement technique requires programs "sequential and their structure must mirror the structure of the abstract model" (as characterized in the Rust-refinement paper). [S82]

---

## 11. Verus - dossier: IronFleet/Verus

- SOSP 2024 abstract: "6.1K lines of implementation and 31K lines of proof" across "distributed systems, an OS page table, a library for NUMA-aware concurrent data structure replication, a crash-safe storage system, and a concurrent memory allocator"; "Verus verifies code 3-61x faster and with less effort than the state of the art." The IronKV delegation map is verified using an "EPR mode" that abstracts the concrete implementation into an EPR model for fully automated invariant proofs. [S58]
- TCB: "Any verification result is limited by its TCB. Verus's results depend on the correctness of the top-level specifications, the Verus verifier, the solvers it relies on, and the Rust compiler." [S58]
- Guide: "At present, we do not intend to: support all Rust features and libraries (instead, we will focus on high-value features and libraries needed to support our users); verify the verifier itself; verify the Rust/LLVM compilers". [S59]
- OOPSLA 2023 extended version, Limitations: Verus "relies on borrow-checking rules and hence cannot reason about traditional Rust unsafe code"; "does not attempt a full aliasing and provenance model for Rust's pointers"; "support for separate verification of multiple crates and functions that return mutable references" missing. [S60]
- RefinedRust (OOPSLA 2026 version) on Verus: "suitable for verifying newly written code... but not for verifying existing code without significant modifications"; Verus "does not support iterators and for loops in general". [S61]

---

## 12. Kani

**What it verifies.** Rust code through proof harnesses; bounded model checking on CBMC.
- Soundness page: "Kani performs bounded model checking. Loops are unwound up to a configurable bound... Verification results are sound only if all unwinding assertions pass." Other caveats on the page: CBMC backend bugs; x87 floating-point differences; pointer object-bit limit default 16; unresolved function pointers over-approximated (spurious failures); `--restrict-vtable` may cause false negatives. [S63]
- Limitations page (fetched in full): a short general statement plus a list of topics, no further detail in the retrieved text. [S64]
- Tool-comparison page: reversing a cryptographic hash with a model checker is easy to try, "but good luck getting it to terminate!"; the page states: "At present, Kani does not support verifying concurrent code." and points to Loom and Shuttle for concurrency. [S65] Rust feature-support page (Yes/Partial/No per Rust Reference section): await expressions are "No" ("See Notes - Concurrency"), patterns and closure/pointer types "Partial"; "No: The feature is not supported. Some support may be available but analyses should not be trusted." [S66]
- Rust std verification experience (Le Blanc and Lam, work in progress): "Kani is a bounded model checker; in this context, the main limitation is that arrays and other data structures are only explored to finite length - this is where the 'small scope hypothesis' comes in"; loops are unrolled with an unwinding assertion or the user supplies an invariant. [S67]
- Kani overview paper (arXiv 2607.01504): Table 1 lists s2n-quic ("Network Protocol", 102 harnesses, 23 min, no contracts) among open-source users. Bug example: in s2n-quic's `try_fit` (how many bytes of a QUIC Stream frame fit given variable-length integer encoding), a differential Bolero harness found no failure in 16,777,216 libfuzzer executions over more than ten minutes, while Kani found a failing assertion in 20 seconds near the one-byte/two-byte integer boundary; a second s2n-quic bug in `decode_packet_number` was found by Kani in 2.8 s and by fuzzing in under a minute. The paper's threats discussion: "Harnesses without loop contracts provide bounded guarantees only; we report unbounded claims only where contracts or passing unwinding assertions justify them" and "three AI-generated contracts required correction (all caught by Kani, not human review)". [S68]
- Scope: these are code-level properties (panic freedom, arithmetic overflow, decoding invariants) of one implementation, not protocol-level properties.

---

## 13. P language

- "P is a state-machine-based language for modeling and analysis of distributed systems" (project site, "What is P?"): "Programmers capture their system design as communicating state machines"; models are meant to be "close to the implementation"; both safety and liveness specifications can be checked. [S71] Case-studies page lists AWS and Microsoft uses (the Windows 8 USB 3.0 stack, S3, EBS, DynamoDB and others). [S70]
- CACM "Systems Correctness Practices at Amazon Web Services" (AWS authors): teams across AWS "have been using P to reason about the correctness of their system designs"; for S3 strong read-after-write consistency P was "used to formally model and validate the protocol design" and "helped eliminate several design-level bugs early"; PObserve (2023) lets teams "take structured logs from the execution of distributed systems and validate post hoc that they match behaviors allowed by the formal P specification", "bridging the gap between the P specification of the system design and the production implementation (typically in languages like Rust or Java)". The same article mentions Lean among the formal methods in use at AWS (for Cedar), alongside TLA+, Dafny and Kani. [S69]
- The PObserve documentation page could not be fetched; limitations of P/PObserve are not documented in the sources read (gap).

---

## 14. Stateright

- GitHub README: "in contrast with model checkers such as TLC for TLA+, systems implemented using Stateright can also be run on a real network without being reimplemented in a different language"; liveness via "eventually" properties is marked experimental/incomplete; includes linearizability and sequential-consistency testers. [S72]
- docs.rs: "unlike many model checkers, Stateright is also able to spawn these actors on a real network." [S73]
- Stateright's own comparison with TLA+ page (full text) lists TLC advantages ("TLC supports arbitrarily complex temporal properties including fairness"; "TLC supports refinement mapping between models") and Stateright's: "With Stateright your model and final implementation are encouraged to share code. This eliminates a possible failure mode whereby a model and it's resulting production system implementation deviate." [S74] The book index describes the actor model and the explicit-state checker. [S75]
- Quint's reference page (secondary): "Actors are plain Rust and can run on a real network, but the system must be built as Stateright actors." [S24]
- Limitations documented in these pages: explicit-state exploration of finite state spaces; experimental liveness; code must be written as Stateright actors. No protocol-scale case study is documented in the fetched pages.

---

## 15. Protocol case-study index

Index of what each protocol case study in this file actually analysed. Depth for each is in the section cited. The dossiers in `research/references/` (Tamarin/ProVerif/CryptoVerif case studies, Ivy QUIC, Verdi, Everest, IronFleet/Verus) cover several of these in more depth.

| Protocol | Study and tool | Object analysed | Reported scope or limit | Section / source |
|---|---|---|---|---|
| TLS 1.3 | Tamarin (Cremers et al. CCS 2017, reported via a secondary description) | Specification model | Tool vendor page reports ~3 months of modeling, >10 GB to load, up to 100 GB to prove, ~1 day to check; the NDSS tutorial slides say "We analyse the specification only" | 1, 4; [S5], [S17] |
| TLS 1.3 | ProVerif (Bhargavan et al.) | Specification model | 1,030 lines, ~70 min, ~3 person-weeks, as reported by Verifpal docs; primary paper not read (lead) | 4; [S17] |
| TLS 1.3 | F* / Everest (record layer, miTLS) | Reference implementation | Record layer proved; handshake implementation proof not completed; 26 F* soundness bugs reported | 9; [S53], [S54] |
| TLS (1.0-era) | Isabelle/HOL inductive method (Paulson) | Protocol model | "six man-weeks of effort and three minutes of processor time" | 8; [S47], [S48] |
| QUIC | Ivy (McMillan-Zuck) | Formal spec used as a tester of real implementations | 27 server errors found; safety only; testing, not proof of the implementations | 6; [S29], [S30], [S31] |
| QUIC | SPIN + ProVerif; ProVerif + Verifpal | Early (Google-QUIC-era) handshake models | Flaws found and fixes proposed; models are of an early design | 15; [S80], [S81] |
| QUIC (implementation) | Kani on s2n-quic | Rust code, bounded harnesses | 102 harnesses in CI in 23 min; bounded guarantees unless loops have contracts | 12; [S68] |
| Signal | Computational pen-and-paper proof (Cohn-Gordon et al.) | Model extracted from the protocol | Authors state they chose some simplicity over a full analysis | 3; [S15] |
| Signal | ProVerif on the Double Ratchet | Protocol model | Three attacks, two confirmed in the main implementation; PCS proof 40 min / 2.7 GB | 2; [S78] |
| Signal | DY* (F*) with ProVerif and CryptoVerif comparison | Protocol code in F* | ProVerif analysis limited to 3 ratcheting rounds (29 h); CryptoVerif 1 round | 15; [S79] |
| WireGuard | Tamarin, CryptoVerif, Noise Explorer (project page) | Models | Project page lists results and properties | 3; [S13], [S12] |
| WireGuard | SAPIC+ with ProVerif and Tamarin | Models | ~15 min for main properties; anonymity ~9 h or ~2 h depending on tool | 15; [S77] |
| WireGuard | Tamarin-to-Go (Arquint et al.) | 608 Go lines linked to Tamarin model | Crypto and network stubs trusted; DoS code, metrics, load balancing removed | 17; [S52] |
| WireGuard | AutoTam | DSL-generated Tamarin models and code | Handshake-phase lemmas only; DoS mitigation, replay window, concurrent transport not implemented | 17; [S76] |
| 5G AKA | Tamarin (Basin et al.) | Specification model | ~500 model lines, 124 lemmas, ~5 h; custom proof strategies needed | 1; [S6], [S7] |
| Raft | Verdi (Coq) | Extracted implementation | Shim layer is trusted; Fonseca et al. found 11 of 16 bugs in shims | 7; [S35], [S36] |
| Raft | TLA+ with trace validation (etcd raft, MongoDB) | Spec versus Go / C++ traces | etcd: spec used to validate the Go library; MongoDB: trace-checking dropped after 10 weeks | 5; [S25], [S27] |
| Paxos / multi-Paxos | Ivy/EPR; Trillium (Iris); IronFleet (Dafny) | Protocol design; Go-like code; Dafny code | EPR: "verify the designs of the protocols and not their implementations"; Trillium: single-decree Paxos refines TLA+ spec; IronFleet 3.6:1 or 7.7:1 proof ratio | 6, 7, 10; [S34], [S43], [S55], [S56] |
| SCION router (not in the question list) | Isabelle/HOL + Gobra | Go router code | 13,400 annotation lines, 2,400 trusted lines, 2.5 person-years; five protocol attacks found | 8; [S51] |

## 16. Limitations by category

Quotes are verbatim from fetched text and each is tied to the source. Categories group the statements by what is being limited. Where a category has no statement from a source, none is implied.

### 16.1 Model-only: the result is about the model, not the code

- Tamarin NDSS tutorial: "We analyse the specification only" and "We assume black-box cryptography" [S5].
- Verifpal docs: a `FAIL` is "not a machine-checkable certificate, and it describes the model rather than an implementation" [S17]. The same page says that moving a model result to code requires "showing that every implementation execution is represented and that satisfying the modeled property implies the intended implementation property" [S17]. Verifpal's quickstart states it does not prove a deployed implementation secure [S19].
- Paxos Made EPR: "We decided to verify the designs of the protocols and not their implementations" [S34].
- Quint reference: the model-to-implementation gap is "closed separately, with conformance testing techniques" [S24].
- Ivy QUIC: the specification is used to test implementations; the paper reports safety properties only [S29], [S30].

### 16.2 Bounded or incomplete algorithms

- Tamarin manual: the procedure is undecidable and may not terminate; observational equivalence is only approximated [S1]. Equations outside subterm-convergent theories can cause infinite loops or incorrect results [S3].
- ProVerif slides: non-termination and false attacks can occur because of abstraction [S9].
- Verifpal: described by its own docs as "bounded and heuristic; no completeness claim"; default is 2 sessions per principal; `PASS [search exhausted at 2 sessions]` can still miss attacks [S17]. The 2026 ePrint abstract describes it as a bounded counterexample finder [S21].
- Apalache: bounded execution plus inductive-invariant checking for fixed parameters [S22]. Quint docs: Apalache "cannot check an arbitrary number of steps" [S23].
- Kani: results are sound only if unwinding assertions pass [S63]. "Harnesses without loop contracts provide bounded guarantees only" [S68]. Based on the small scope hypothesis [S67].
- Stateright: liveness checking is experimental [S72].

### 16.3 Trusted computing base and assumptions

- Ivy: "Ivy relies on a vastly larger trusted computing base than typical proof assistants" [S31].
- Verus: the TCB is the specification, the verifier, the SMT solvers and rustc [S58].
- IronFleet: the specification and a trusted UDP specification are trusted [S55].
- Grove: network and filesystem libraries are trusted [S44].
- Dafny: `assume` "may lead to invalid conclusions"; "By supplying an {:extern} or {:axiom} attribute, you are taking that responsibility into your own hands" [S62].
- HMAC-DRBG (VST): result holds "assuming that the DRBG's initial state is ideally random" [S39].
- Tamarin-to-Go (Arquint et al.): crypto and network stubs are trusted [S52].
- Everest retrospective: 26 soundness bugs in F* itself were found over the project [S53].

### 16.4 Concurrency, liveness and property classes

- Kani: "At present, Kani does not support verifying concurrent code" [S65]. Await expressions are listed as unsupported [S66].
- Verus: cannot reason about traditional Rust unsafe code [S60].
- Grove: "Grove is only able to verify safety properties" [S44]. Fairneris adds liveness under network fairness for a case study [S46].
- Trillium: IronFleet "does not support node-local concurrency"; Igloo "proves a particular kind of extensional safety properties" [S43].
- Arquint et al.: equivalence (indistinguishability) properties are an open problem for the approach [S52].
- SoK on consensus verification: "liveness verification remains underdeveloped" and "scalability limits restrict verification to small networks" [S83].

### 16.5 Language and feature coverage ("any codebase")

- Verus: "we do not intend to support all Rust features" [S59]; the RefinedRust paper characterises Verus-style approaches as not for verifying existing code without significant modifications [S61].
- Kani publishes a Rust feature-support table with Yes/Partial/No entries [S66].
- Dafny verified code is compiled to a target language (see [S62] for targets); the IronFleet release is flagged experimental and built with Dafny 3.4.0 and .NET 6.0 [S57].
- Tamarin-to-Go targets Go (Gobra) and Java (VeriFast) code [S52]; Igloo targets Java and Python through VeriFast and Nagini [S50].
- Noise Explorer generates code for specific Noise patterns [S16].
- AutoTam requires implementations written in its DSL [S76].

### 16.6 Effort and scale

- IronFleet: 3.7 person-years and 3.6:1 proof-to-code ratio in the SOSP paper [S55]; 7.7:1 (5.4:1 without liveness) in the CACM version [S56].
- Grove: 2,435 verified Go lines against 28,077 Coq specification and proof lines, a 12x ratio, and 1,597 lines for new reasoning principles [S44].
- Verus: 6.1K implementation lines against 31K proof lines in one summary figure and a "3-61x" proof-to-code range [S58].
- SCION router: 13,400 annotation lines and 2.5 person-years [S51].
- Tamarin TLS 1.3: roughly 3 months modeling and up to 100 GB of memory to prove (secondary report) [S17]. Tamarin 5G: ~1,000 lines of Python proof strategies on top of 124 lemmas [S17].
- Paulson on TLS: six man-weeks [S47]. Disk Paxos in Isabelle: ~6,500 lines [S34].

### 16.7 Tool soundness and implementation bugs

- Verifpal docs report soundness-audit findings in ProVerif and Tamarin, including false `verified` results on Tamarin 1.12.0 [S17] (vendor-authored, see Conflicts).
- Fonseca et al.: 16 bugs in verified distributed systems, 11 in shim layers, none in protocol logic [S35].
- Everest: 26 F* soundness bugs [S53].

## 17. Model-to-code linking frameworks

How each documented approach connects a protocol model to code. This section describes mechanisms; it does not rank them.

| Framework | Model side | Code side | Link mechanism as documented | Gap statements | Source |
|---|---|---|---|---|---|
| Arquint et al. | Tamarin (MSR rules) | Go via Gobra; Java via VeriFast | Model translated to per-role I/O specs; Theorem 2 gives trace inclusion between code and model | Crypto and network stubs trusted; WireGuard code reduced (DoS, metrics, load balancing removed); equivalence properties open | [S52] |
| AutoTam | Generated Tamarin models | DSL-based implementation, symbolic execution for memory safety | DSL is the single source for model and code | Handshake-phase lemmas only for WireGuard | [S76] |
| Igloo + SCION router | Isabelle/HOL refinement | Go (Gobra), Java (VeriFast), Python (Nagini) | Refinement proof linked to separation-logic code proofs | 2,400 trusted lines in the router; safety properties | [S50], [S51] |
| cv2fstar | CryptoVerif | F* specs on HACL* | Compiler from CryptoVerif models to F* | Thesis names the computational-proof-to-code gap | [S14] |
| DY* | Symbolic model written in F* | Executable F* protocol code | Model and protocol code written in the same language (F*) | Signal analysis; F* code, not arbitrary languages | [S79] |
| Noise Explorer | ProVerif models | Go and Rust code per Noise pattern | Generated from one pattern description | Pattern-limited | [S16] |
| Trillium (Iris/Coq) | TLA+-style abstract spec | Concrete AnerisLang programs | Refinement of traces in Iris | Safety refinement; 2PC and single-decree Paxos | [S43] |
| Grove (Iris/Coq) | Specification of services | Go code | Separation logic over Go programs | Network and filesystem libraries trusted; safety only | [S44] |
| Rust refinement methodology | First-order guarded transition systems | Rust, SMT-verified | Locally inductive invariants relate model to concrete state | Contrasts itself with Iris (manual) and IronFleet (structure restrictions) | [S82] |
| IronFleet / Verus | Dafny / Verus ghost specs | Dafny-compiled code / Rust | Layered refinement; Verus embeds model as ghost state | TCB listed in section 16.3; Verus EPR mode used for IronKV delegation map | [S55], [S58] |
| TLA+ trace validation (etcd, MongoDB) | TLA+ | Go / C++ | Implementation traces checked against spec | MongoDB trace checking dropped after 10 weeks; later test generation reached 100% branch coverage | [S25], [S27], [S28] |
| P with PObserve | P state machines | Service logs | Post-hoc log validation "bridging the gap" | PObserve documentation page not read (lead) | [S69] |
| Stateright | Rust actor model | Same actors on a real network | Model and implementation encouraged to share code | System must be built as Stateright actors | [S74], [S24] |

## 18. Source reliability notes

- Vendor-authored comparisons: [S17] is written by the developer of Verifpal and compares against ProVerif and Tamarin. Its effort figures for TLS 1.3 and 5G are second-hand reports of other papers; the primary papers for TLS 1.3 were not read in this phase.
- [S21] is an ePrint abstract page; the full paper was not read.
- [S27] is an industry blog; [S28] is the peer-reviewed counterpart.
- [S41] is a press article; [S53] and [S40] are the primary sources for Everest and Fiat.
- [S81] is hosted on a mirror (scispace); the underlying paper venue was not confirmed. Treat as secondary.
- [S23], [S24], [S32], [S49] are secondary (tool-vendor or practitioner descriptions of other tools).
- Several recent arXiv and conference items ([S10], [S46], [S61], [S68], [S76], [S82], [S83]) were read as single sources and not cross-checked against independent ones.
- Aggregator and hobby sites that appeared in searches were not used (listed under Unverified leads).

## 19. Conflicts

1. IronFleet proof-to-code ratio. [S55] reports a 3.6:1 ratio; [S56] (same authors, later article) reports 7.7:1 (5.4:1 without liveness) with a table of line counts (spec 1,400; implementation 5,114; proof 39,253; 395 minutes). The two papers appear to count differently; I did not reconcile them. Unresolved.
2. Verifpal session bounds. [S17] and [S21] describe Verifpal as bounded with a default of 2 sessions. A search snippet from the 2020 Verifpal paper, seen in Phase 1 but not read in full this phase, suggested an unbounded-session claim and a Coq semantics. Because that paper is a lead only, the conflict is recorded but not resolved.
3. miTLS verification status. Aggregator pages (leads, not read as evidence) describe miTLS as verified against RFC 8446; the Everest retrospective [S53] states the handshake implementation proof was not completed. The retrospective is the primary source; the aggregator claim is unverified. Unresolved pending a primary miTLS source for the handshake.
4. Tool soundness. [S17] reports false `verified` results on Tamarin 1.12.0 and other audit findings; the Tamarin and ProVerif documentation I read ([S1], [S9]) describes their own limitations but does not respond to those findings. Maintainer response not found.
5. Meaning of "refinement of an implementation". [S43] describes Trillium's result as the first foundationally verified proof that a concrete implementation refines an abstract TLA+ specification, while IronFleet [S55] and Igloo/SCION [S50], [S51] also link specifications to code with different trust bases (SMT-based or mixed provers). The sources use different definitions of "verified" and "foundational"; no common standard exists across them.
6. Verification versus testing. [S30] uses a formal Ivy specification to test implementations and reports errors found, while [S31] describes Ivy's proof-oriented capabilities and trusted base. Whether this counts as verifying an implementation depends on the definition (D); sources do not agree on one.

## 20. Gaps closed and gaps remaining

Closed since Phase 1 (all by fetching full text of primary sources):
- Verifpal, Apalache, mypyvy, VST, Iris (Aneris, Trillium, Grove, Fairneris), Isabelle (Paulson, SCION), Kani, P, Stateright and Dafny reference all now have full-read entries.
- A limitations-by-category section (16) with verbatim quotes.
- Model-to-code frameworks (17).

Remaining:
- Primary text of Bhargavan et al. S&P 2017 (TLS 1.3 in ProVerif) was not obtained; the numbers come from [S17].
- No P or PObserve limitations page was found.
- Dafny tutorial and FAQ were not fetched; only the reference manual [S62] was read.
- Long PDFs were read by section and keyword, not line by line.
- Kani-on-protocol evidence is limited to the s2n-quic cases in [S68].
- Verifpal's 2020 paper was not fetched (see Conflict 2).
- No independent replication of any claimed proof effort was attempted.

## 21. Open Questions

1. D: What does each community mean by "verify an implementation" (functional correctness, safety refinement, memory safety, conformance testing, trace inclusion), and is there a shared definition across the sources in sections 15-17?
2. H3: For TLS 1.3, QUIC, Signal and WireGuard, which public artefacts link a protocol-level proof to a mainstream-language implementation, and which parts remain trusted in each?
3. H2: How do proof effort and runtime figures (sections 16.6) scale for protocols larger than those studied?
4. H4: Which tools in this file handle an arbitrary existing codebase without rewriting it in a DSL, a verified subset or a specific language?
5. H1: Which protocol properties (liveness, equivalence, computational security) are covered by which tool, and where do the sources state they are not?
6. H5: Do any sources argue that a limit of a tool on one side (model or code) implies anything about the other side?
7. Does the Verifpal 2020 unbounded-session claim hold, and how does it relate to the 2026 description of bounded search (Conflict 2)?
8. What is the final status of the miTLS handshake proof (Conflict 3)?
9. Would a primary source for PObserve describe its limits on conformance checking of production logs?
10. Are there proof-assistant protocol analyses (Isabelle scyther-proof, DyLean, noted in [S17] as using Isabelle and Lean) that should be read as primary sources for the Lean-related sub-claims? This is handed to the agents covering Lean.

## 22. Unverified leads (excluded from evidence)

Items found only through search snippets, failed fetches, paywalls or aggregators. None is cited as evidence above except where a conflict is flagged as unresolved.

1. Bhargavan et al., "Verified Models and Reference Implementations for TLS 1.3" and the S&P 2017 ProVerif TLS 1.3 paper (fetch failed).
2. Cremers et al., CCS 2017 Tamarin TLS 1.3 paper (only secondary figures in [S17]).
3. Verifpal 2020 Springer paper.
4. Aneris ESOP 2020 paper (project page [S42] read instead).
5. Verdi PLDI 2015 paper (fetch failed; opam page [S36] read instead).
6. Velisarios (Springer, paywalled).
7. Apalache vs TLC Springer 2026 chapter.
8. Inria Apalache report.
9. Specula arXiv preprint.
10. QUICstep Tamarin paper.
11. PObserve documentation.
12. hax / hacspec (Rust to F* / ProVerif).
13. Charron-Bost and Merz, Heard-Of Paxos.
14. MIT 6.826 lecture notes on Everest.
15. Cortier et al., CryptoVerif dynamic key compromise paper.
16. Aggregators and hobby sites (sota.io, positive-intentions, emergentmind, quant67, tractable.for-all.dev).


## Source Register

All entries: access date 2026-11-07, depth [FULL] (see "How to read" for what full-read means for long PDFs). Tags: [P] primary, [S] secondary. Sub-claim tags show relevance only.

- **S1** Tamarin manual: Limitations. <https://tamarin-prover.com/manual/master/book/014_limitations.html> | 2026-11-07 | [P] | [FULL] | H1 H2 D. Symbolic model only; subterm-convergent theories beyond builtins; undecidable, may not terminate; observational equivalence only approximated.
- **S2** Tamarin manual: Introduction. <https://tamarin-prover.com/manual/master/book/001_introduction.html> | 2026-11-07 | [P] | [FULL] | H2 D. Result is a proof for unbounded role instances or a counterexample, if search terminates.
- **S3** Tamarin manual: Modeling issues. <https://tamarin-prover.com/manual/master/book/010_modeling-issues.html> | 2026-11-07 | [P] | [FULL] | H1 H2. Non-subterm-convergent equations can cause infinite loops or incorrect results.
- **S4** Tamarin prover home page. <https://tamarin-prover.com/index.html> | 2026-11-07 | [P] | [FULL] | H1. Lists multiset-rewriting input, protocol case studies (Noise, EMV, iMessage and others).
- **S5** NDSS 2017 tutorial slides: Automated analysis of TLS 1.3 with Tamarin. <https://www.ndss-symposium.org/wp-content/uploads/2017/09/T3-tamarin.pdf> | 2026-11-07 | [P] | [FULL] | H3 D. Slides state 'We analyse the specification only' and 'We assume black-box cryptography'.
- **S6** Basin et al., A Formal Analysis of 5G Authentication (CCS 2018). <https://arxiv.org/pdf/1806.10360> | 2026-11-07 | [P] | [FULL] | H1 H2. Tamarin model of 5G AKA; ~500 lines, 124 lemmas, ~5 hours; fully automatic mode fails without custom proof strategies.
- **S7** Cremers: Tamarin 5G AKA page. <https://people.cispa.io/cas.cremers/tamarin/5G/> | 2026-11-07 | [P] | [FULL] | H3. Vulnerability in Feb 2018 draft TS 33.501; page does not state an implementation was analysed.
- **S8** ProVerif home page. <https://bblanche.gitlabpages.inria.fr/proverif/> | 2026-11-07 | [P] | [FULL] | H1 H2. Tool description: symbolic Dolev-Yao protocol verifier; lists protocols analysed.
- **S9** Blanchet, ProVerif slides (Secret10). <https://bblanche.gitlabpages.inria.fr/talks/Secret10.pdf> | 2026-11-07 | [P] | [FULL] | H2. Lists limitations: non-termination, false attacks; abstraction ignores repetition.
- **S10** Delaune, 25 Years of ProVerif (LICS 2026 invited paper). <https://drops.dagstuhl.de/storage/00lipics/lipics-vol380-lics2026/LIPIcs.LICS.2026.2/LIPIcs.LICS.2026.2.pdf> | 2026-11-07 | [P] | [FULL] | H1 H2 H3. Survey of the tool's history; names usability and interoperability between verification approaches as ongoing challenges.
- **S11** CryptoVerif home page. <https://bblanche.gitlabpages.inria.fr/cryptoverif/> | 2026-11-07 | [P] | [FULL] | H1 H2 H3. Computational-model prover; polynomial sessions; lists translations to OCaml and F*.
- **S12** CryptoVerif WireGuard page. <https://bblanche.gitlabpages.inria.fr/cryptoverif/WireGuard/> | 2026-11-07 | [P] | [FULL] | H2. Model files and scripts for the WireGuard computational proof.
- **S13** WireGuard formal verification page. <https://www.wireguard.com/formal-verification/> | 2026-11-07 | [P] | [FULL] | H2 H3. Lists Tamarin, CryptoVerif, Noise Explorer results and the properties covered.
- **S14** Lipp, PhD thesis (cv2fstar). <https://theses.hal.science/tel-03933719/> | 2026-11-07 | [P] | [FULL] | H3. Names the gap between computational proofs and implementations; compiler from CryptoVerif to F* specs on HACL*.
- **S15** Cohn-Gordon et al., A Formal Security Analysis of the Signal Messaging Protocol. <https://eprint.iacr.org/2016/1013.pdf> | 2026-11-07 | [P] | [FULL] | H1 H3. Pen-and-paper computational proof of X3DH + Double Ratchet core; model extracted from implementation; limitations section on omitted features.
- **S16** Kobeissi, Nicolas, Bhargavan, Noise Explorer. <https://eprint.iacr.org/2018/766.pdf> | 2026-11-07 | [P] | [FULL] | H1 H3. Generates ProVerif models and Go/Rust implementations for Noise patterns; 57+ patterns analysed.
- **S17** Verifpal: Comparison. <https://verifpal.com/docs/comparison/> | 2026-11-07 | [P] | [FULL] | H1 H2 H3 D. Vendor comparison of Verifpal, ProVerif, Tamarin: tradeoff triangle, result codes, effort figures for TLS 1.3 and 5G studies, audit of tool soundness, model-to-code obligations. Authored by Verifpal's developer.
- **S18** Verifpal: Language reference. <https://verifpal.com/docs/reference/> | 2026-11-07 | [P] | [FULL] | H1. Language and query reference (read for result semantics).
- **S19** Verifpal: Quickstart. <https://verifpal.com/docs/> | 2026-11-07 | [P] | [FULL] | H3 D. States it does not prove a deployed implementation secure.
- **S20** Verifpal: Introduction. <https://verifpal.com/docs/introduction/> | 2026-11-07 | [P] | [FULL] | H1 H3. Model rules omit byte lengths, timing, implementation bugs, computational probabilities.
- **S21** Kobeissi, From Toy to Instrument: Seven Years of Verifpal (ePrint 2026/1654). <https://eprint.iacr.org/2026/1654> | 2026-11-07 | [P] | [FULL] | H2 D. Abstract of evaluation: Verifpal as bounded counterexample finder; a pass is not an unbounded proof; no report is a machine-checked certificate. Abstract page only, paper PDF not read.
- **S22** Konnov, Kukovec, Tran, TLA+ Model Checking Made Symbolic (OOPSLA 2019). <https://hal.science/hal-02280888/document> | 2026-11-07 | [P] | [FULL] | H2 D. Apalache: SMT-based bounded and inductive checking of TLA+, finite parameters; evaluation versus TLC.
- **S23** Quint docs: Model checkers. <https://quint.sh/docs/model-checkers> | 2026-11-07 | [S] | [FULL] | H2 D. TLC explicit-state; Apalache symbolic bounded, cannot check arbitrary number of steps.
- **S24** Quint: Reference of formal verification tools. <https://quint.sh/guides/formal-verification-tools> | 2026-11-07 | [S] | [FULL] | H3. Spec-to-code gap closed separately by conformance testing; Stateright described as actors on a real network.
- **S25** etcd-io/raft tla/README.md. <https://github.com/etcd-io/raft/blob/main/tla/README.md> | 2026-11-07 | [P] | [FULL] | H3. TLA+ spec of the Raft library used for model checking and for trace validation of the Go implementation.
- **S26** tracevalidation/Raft repository. <https://github.com/tracevalidation/Raft> | 2026-11-07 | [P] | [FULL] | H3. Raft TLA+ trace-validation repository (research artifact; 1 star, no description).
- **S27** MongoDB engineering blog: Conformance checking at MongoDB. <https://www.mongodb.com/company/blog/engineering/conformance-checking-at-mongodb-testing-our-code-matches-our-tla-specs> | 2026-11-07 | [P] | [FULL] | H3 D. Trace checking abandoned after 10 weeks; traces never matched the spec.
- **S28** Davis et al., eXtreme Modelling in Practice (VLDB 2020). <http://www.vldb.org/pvldb/vol13/p1346-davis.pdf> | 2026-11-07 | [P] | [FULL] | H3 D. MBTC impractical for MongoDB server; test-case generation for Realm OT reached 100% branch coverage and found a bug.
- **S29** Ivy QUIC specification README. <https://github.com/microsoft/ivy/blob/master/doc/examples/quic/README.md> | 2026-11-07 | [P] | [FULL] | H1 H3. States spec limits: subset of QUIC features, no quantitative time.
- **S30** McMillan and Zuck, Formal Specification and Testing of QUIC (SIGCOMM 2019). <https://dl.acm.org/doi/pdf/10.1145/3341302.3342087> | 2026-11-07 | [P] | [FULL] | H1 H3 D. Ivy spec as executable monitor; 27 server errors in four implementations; safety only; testing not proof.
- **S31** Ivy CAV 2020 tool paper. <https://pmc.ncbi.nlm.nih.gov/articles/PMC7363183/> | 2026-11-07 | [P] | [FULL] | H2 D. Ivy compared with proof assistants: more automation, vastly larger trusted base, no reflection.
- **S32** Galois blog: QUIC testing. <https://www.galois.com/articles/quic-testing-a-quick-replication> | 2026-11-07 | [S] | [FULL] | H3. Vendor summary of the Ivy QUIC testing work.
- **S33** mypyvy README. <https://github.com/wilcoxjay/mypyvy> | 2026-11-07 | [P] | [FULL] | H1 H2. Language for symbolic transition systems; invariant checking and inference; input is a transition system.
- **S34** Padon et al., Paxos Made EPR (arXiv 1710.07191). <https://arxiv.org/pdf/1710.07191> | 2026-11-07 | [P] | [FULL] | H1 H2 H3. Paxos family safety proofs in a decidable fragment; authors verify designs, not implementations; safety only.
- **S35** Fonseca et al., An Empirical Study on the Correctness of Formally Verified Distributed Systems (EuroSys 2017). <https://cs.purdue.edu/homes/pfonseca/papers/eurosys2017-dsbugs.pdf> | 2026-11-07 | [P] | [FULL] | H3 H5. 16 bugs in IronFleet, Verdi, Chapar; 11 in shim layers; none in protocol logic.
- **S36** verdi-raft opam package. <https://github.com/uwplse/verdi-raft/blob/master/coq-verdi-raft.opam> | 2026-11-07 | [P] | [FULL] | H3. Describes the verified Raft implementation in Coq built with Verdi.
- **S37** Verified Software Toolchain site. <https://vst.cs.princeton.edu/> | 2026-11-07 | [P] | [FULL] | H3. Machine-checked proofs down to machine language for C programs.
- **S38** Beringer et al., Verified Correctness and Security of OpenSSL HMAC (USENIX Security 2015). <https://www.usenix.org/system/files/conference/usenixsecurity15/sec15-paper-beringer.pdf> | 2026-11-07 | [P] | [FULL] | H3. VST + FCF + CompCert: C implementation meets FIPS functional spec; spec has crypto properties.
- **S39** Appel et al., Verified Correctness and Security of mbedTLS HMAC-DRBG. <https://www.cs.princeton.edu/~appel/papers/verified-hmac-drbg.pdf> | 2026-11-07 | [P] | [FULL] | H3. Crypto security of the spec plus correctness of the C code, assuming ideally random initial state.
- **S40** Erbsen et al., Simple High-Level Code For Cryptographic Arithmetic (SP 2019). <https://adam.chlipala.net/papers/FiatCryptoSP19/FiatCryptoSP19.pdf> | 2026-11-07 | [P] | [FULL] | H3. Verified generator for field arithmetic; code in BoringSSL for Chrome/Android/CloudFlare.
- **S41** MIT News on Fiat Cryptography. <https://news.mit.edu/2019/fiat-cryptography-chrome-android-0617> | 2026-11-07 | [S] | [FULL] | H3. Press claim that ~90 percent of secure Chrome communications run Fiat-generated code.
- **S42** Aneris project page. <https://iris-project.org/aneris/> | 2026-11-07 | [P] | [FULL] | H1 H3. Higher-order distributed concurrent separation logic in Coq.
- **S43** Timany et al., Trillium (POPL 2024). <https://iris-project.org/pdfs/2024-popl-trillium.pdf> | 2026-11-07 | [P] | [FULL] | H1 H2 H3. Intensional refinement logic; 2PC and single-decree Paxos refine TLA+ specs; comparison notes on IronFleet and Igloo.
- **S44** Sharma et al., Grove (SOSP 2023). <https://pdos.csail.mit.edu/papers/grove:sosp23.pdf> | 2026-11-07 | [P] | [FULL] | H1 H2 H3. Verified Go distributed KV store; 2,435 verified Go lines, 28,077 Coq lines; safety only.
- **S45** Timany, Modular Verification of Distributed Systems (PLaID 2025 talk). <https://cs.au.dk/~timany/talks/files/2025_PLaID.pdf> | 2026-11-07 | [P] | [FULL] | H1 H3. Slides listing systems verified with Aneris.
- **S46** Fairneris: higher-order separation logic for liveness (preprint). <https://hal.science/hal-05647709/document> | 2026-11-07 | [P] | [FULL] | H1 H2. Liveness for higher-order concurrent distributed programs under network fairness; one RPC case study.
- **S47** Paulson, Inductive analysis of the Internet protocol TLS (TR 440). <https://www.cl.cam.ac.uk/techreports/UCAM-CL-TR-440.pdf> | 2026-11-07 | [P] | [FULL] | H1 H2. Isabelle/HOL analysis of TLS; six man-weeks, three minutes of processor time.
- **S48** Paulson, The Inductive Approach to Verifying Cryptographic Protocols (JCS 1998). <https://www.cl.cam.ac.uk/~lp15/papers/Auth/jcs.pdf> | 2026-11-07 | [P] | [FULL] | H1 H2. Inductive trace-set method in Isabelle/HOL; no finite-state restriction.
- **S49** Kleppmann, Verifying distributed systems with Isabelle/HOL. <https://martin.kleppmann.com/2022/10/12/verifying-distributed-systems-isabelle.html> | 2026-11-07 | [S] | [FULL] | H1 H2. Practitioner blog on modeling distributed algorithms in Isabelle/HOL.
- **S50** Igloo (OOPSLA 2020) conference page. <https://2020.splashcon.org/details/splash-2020-oopsla/28/Igloo-Soundly-Linking-Compositional-Refinement-and-Separation-Logic-for-Distributed-> | 2026-11-07 | [P] | [FULL] | H3. Isabelle/HOL refinement framework linked to VeriFast/Nagini code verification; abstract only.
- **S51** Formal Verification of a Next-Generation Internet Router (SCION). <https://www.research-collection.ethz.ch/server/api/core/bitstreams/bcc7d4a8-ec35-4db3-8eb5-9d92420b2500/content> | 2026-11-07 | [P] | [FULL] | H1 H2 H3. Isabelle/HOL refinement of SCION protocol linked to Gobra-verified Go router code; 2.5 person-years; five protocol attacks found.
- **S52** Arquint et al., Sound Verification of Security Protocols: From Design to Interoperable Implementations. <https://arxiv.org/pdf/2212.04171> | 2026-11-07 | [P] | [FULL] | H3 H5. Tamarin model to per-role I/O specs to Gobra/VeriFast code proofs; WireGuard Go case study.
- **S53** Everest TOPLAS retrospective. <https://dl.acm.org/doi/full/10.1145/3805702> | 2026-11-07 | [P] | [FULL] | H2 H3. TLS 1.3 handshake implementation proof not completed; record layer, HACL*, EverParse verified; 26 F* soundness bugs.
- **S54** Delignat-Lavaud et al., Implementing and Proving the TLS 1.3 Record Layer. <https://www.microsoft.com/en-us/research/wp-content/uploads/2017/08/191.pdf> | 2026-11-07 | [P] | [FULL] | H3. F* reference implementation of record layer with reduction to cipher assumptions.
- **S55** Hawblitzel et al., IronFleet (SOSP 2015). <https://www.microsoft.com/en-us/research/wp-content/uploads/2015/10/ironfleet.pdf> | 2026-11-07 | [P] | [FULL] | H2 H3. Three-layer Dafny refinement; 3.6:1 proof ratio; 3.7 person-years; trusted spec and UDP layer.
- **S56** Hawblitzel et al., Proving Safety and Liveness of Practical Distributed Systems (CACM 2017). <https://www.microsoft.com/en-us/research/wp-content/uploads/2017/06/ironfleet-cacm.pdf> | 2026-11-07 | [P] | [FULL] | H2 H3. Revised IronFleet account; 7.7:1 ratio; code-size table.
- **S57** Ironclad repository README. <https://github.com/microsoft/Ironclad/blob/main/ironfleet/README.md> | 2026-11-07 | [P] | [FULL] | H3 H4. Released IronFleet code is experimental, built with Dafny 3.4.0 and .NET 6.0.
- **S58** Lattuada et al., Verus: A Practical Foundation for Systems Verification (SOSP 2024). <https://verus-lang.github.io/paper-sosp24-artifact/assets/paper-20240921-162720-b7db935.pdf> | 2026-11-07 | [P] | [FULL] | H2 H3. 6.1K impl / 31K proof lines across systems; TCB statement; EPR mode for IronKV.
- **S59** Verus guide. <https://verus-lang.github.io/verus/guide/> | 2026-11-07 | [P] | [FULL] | H4. States non-goals: not all Rust features, no verified verifier or compiler.
- **S60** Lattuada et al., Verus: Verifying Rust Programs using Linear Ghost Types (OOPSLA 2023, extended). <https://arxiv.org/html/2303.05491> | 2026-11-07 | [P] | [FULL] | H4. Limitations: no traditional unsafe, no full aliasing model, missing features.
- **S61** RefinedRust (OOPSLA 2026 version). <https://iris-project.org/pdfs/2026-oopsla-refinedrust2.pdf> | 2026-11-07 | [P] | [FULL] | H4. Related-work remarks on Verus and Creusot/Prusti applicability to existing code.
- **S62** Dafny Reference Manual. <https://dafny.org/latest/DafnyRef/DafnyRef> | 2026-11-07 | [P] | [FULL] | H3 D. assume, {:axiom}, {:extern} shift proof burden to the user; compilation targets.
- **S63** Kani: Soundness. <https://model-checking.github.io/kani/soundness.html> | 2026-11-07 | [P] | [FULL] | H2 D. Results sound only if unwinding assertions pass; list of caveats.
- **S64** Kani: Limitations. <https://model-checking.github.io/kani/limitations.html> | 2026-11-07 | [P] | [FULL] | H4. Short page with a general statement and topic list.
- **S65** Kani: Comparison with other tools. <https://model-checking.github.io/kani/tool-comparison.html> | 2026-11-07 | [P] | [FULL] | H2 H4. No concurrent code verification; hash reversal 'good luck' remark.
- **S66** Kani: Rust feature support. <https://model-checking.github.io/kani/rust-feature-support.html> | 2026-11-07 | [P] | [FULL] | H4. Yes/Partial/No table of Rust Reference features.
- **S67** Le Blanc and Lam, Lessons Learned from a Community Effort to Verify the Rust Standard Library. <https://arxiv.org/pdf/2510.01072v3> | 2026-11-07 | [P] | [FULL] | H2 H4. Kani as bounded model checker; small scope hypothesis; loops need unwinding assertions or invariants.
- **S68** Kani: A Model Checker for Rust (arXiv 2607.01504). <https://arxiv.org/html/2607.01504> | 2026-11-07 | [P] | [FULL] | H2 H3. Architecture and evaluation; s2n-quic bug findings, 102 harnesses in CI; threats section.
- **S69** Systems Correctness Practices at Amazon Web Services (CACM). <https://cacm.acm.org/practice/systems-correctness-practices-at-amazon-web-services/> | 2026-11-07 | [P] | [FULL] | H3 D. AWS use of P, TLA+, Dafny, Kani and Lean; PObserve log conformance.
- **S70** P: Case studies. <https://p-org.github.io/P/casestudies/> | 2026-11-07 | [P] | [FULL] | H3. Lists Microsoft USB driver, AWS and other uses.
- **S71** P: What is P?. <https://p-org.github.io/P/whatisP/> | 2026-11-07 | [P] | [FULL] | H1 H3. State-machine language, close to implementation, safety and liveness specs.
- **S72** Stateright GitHub README. <https://github.com/stateright/stateright> | 2026-11-07 | [P] | [FULL] | H3. Actors can be model checked and run on a real network; liveness experimental.
- **S73** Stateright docs.rs. <https://docs.rs/stateright/latest/stateright/> | 2026-11-07 | [P] | [FULL] | H3. API docs; real-network actor runtime.
- **S74** Stateright: Comparison with TLA+. <https://www.stateright.rs/comparison-with-tlaplus.html> | 2026-11-07 | [P] | [FULL] | H3 D. TLC and Stateright advantages; shared model and implementation code.
- **S75** Stateright book. <https://www.stateright.rs/> | 2026-11-07 | [P] | [FULL] | H1. Book index for actor model and checker.
- **S76** Wilson et al., AutoTam (arXiv 2606.19937). <https://arxiv.org/html/2606.19937v1> | 2026-11-07 | [P] | [FULL] | H3. DSL for protocol implementations, generates Tamarin models, symbolic execution for memory safety; WireGuard without DoS/replay window/concurrency.
- **S77** A Unified Symbolic Analysis of WireGuard (NDSS 2024). <https://www.ndss-symposium.org/wp-content/uploads/2024-364-paper.pdf> | 2026-11-07 | [P] | [FULL] | H2. SAPIC+ with ProVerif and Tamarin; run times for agreement, secrecy, anonymity.
- **S78** Automated formal analysis of Signal's Double Ratchet (HAL 2026). <https://hal.science/hal-05589499/document> | 2026-11-07 | [P] | [FULL] | H2 H3. ProVerif analysis of all Double Ratchet features; three attacks, two confirmed in main implementation.
- **S79** Bhargavan et al., DY*: A Modular Symbolic Verification Framework for Executable Cryptographic Protocol Code (EuroS&P 2021). <https://inria.hal.science/hal-03178425/file/dy-star-eurosp21.pdf> | 2026-11-07 | [P] | [FULL] | H2 H3. F* framework for symbolic analysis of protocol code; Signal case study; remarks on prior tool limits.
- **S80** A Systematic Approach to Formal Analysis of QUIC Handshake (Wiley, 2021). <https://onlinelibrary.wiley.com/doi/10.1155/2021/1630223> | 2026-11-07 | [P] | [FULL] | H1. SPIN + ProVerif analysis of early QUIC handshake; design flaw and proposed fix.
- **S81** Formal Analysis of QUIC Handshake Protocol Using Symbolic Model Checking (mirror). <https://scispace.com/pdf/formal-analysis-of-quic-handshake-protocol-using-symbolic-3numg0riuw.pdf> | 2026-11-07 | [S] | [FULL] | H1. ProVerif and Verifpal on an early QUIC handshake; ProVerif stronger on authentication. Hosted on a mirror site.
- **S82** A Refinement Methodology for Distributed Programs in Rust (ACM 3763119). <https://dl.acm.org/doi/10.1145/3763119> | 2026-11-07 | [P] | [FULL] | H3. Bottom-up refinement with locally inductive invariants, SMT-based Rust verifier; contrasts with Iris and IronFleet.
- **S83** SoK: Formal Verification of Consensus Protocols (arXiv 2608.21935). <https://arxiv.org/pdf/2608.21935> | 2026-11-07 | [P] | [FULL] | H2 H3. Maturity scale; gaps: liveness, spec-implementation disconnect, scalability.
