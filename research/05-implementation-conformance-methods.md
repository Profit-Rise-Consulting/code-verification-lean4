# 05 Implementation Conformance Methods

Counts: [FULL] 10 entries (A1-A5, B1-B2, C1-C3), [CODE] 9 entries (D1-D9), unverified leads 39 (listed at the end, excluded from evidence).
Phase 1b rewrite, access date for all entries: 2026-11-07. No analysis, no verdict.

## Conventions used here
- Tags: [P] primary, [S] secondary. Depth: [FULL] means I obtained the full text and read the sections named in "Read" (not every section of every paper); [CODE] means I cloned the repository (depth 1, no git history kept) and inspected its docs and source.
- Sub-claim IDs follow research/CONVENTIONS.md: H1 expressiveness, H2 provability, H3 implementation link (spec to code with a stated guarantee), H4 generality (any codebase), H5 inference (implementations checked against a spec without protocol-level proofs), D definitions of "verify" and "check". Non-Lean methods in Parts A to C are tagged by what they show about D, H4 and H5; where a source compares code to a formal or RFC-derived spec they are also tagged H3 as a non-Lean point of comparison.
- Lean counts come from a comment- and string-stripped regular-expression scan (script not committed; counts are approximate). `axiom` and `opaque` count declarations at line start; `extern` counts attributes; `partial` counts `partial def`.
- Dossier agents cover Cedar differential testing, Ivy QUIC testing and TLA+ trace validation (CCF, MongoDB) under research/references/. Those topics are not re-read here; see "Pointers" below.

## Part A. Active automata learning of protocol implementations

### A1. Protocol state fuzzing of TLS implementations (de Ruiter and Poll, USENIX Security 2015) [P] [FULL]
- URL: https://www.cs.ru.nl/~erikpoll/papers/tls_fuzzing.pdf
- Bears on: D, H3 (non-Lean comparison), H4, H5.
- Read (full text obtained; the parts named here were read, other parts only skimmed): abstract, Sections 1 to 5 (method, alphabet, per-implementation findings), related work.
- Method: LearnLib-style Mealy-machine learning with a test harness that sends abstract TLS messages; equivalence queries approximated with a modified W-method. Table 2 reports, for a regular alphabet with the modified W-method at depth 2, 9 minutes for a 7-state model; with the full alphabet, 27 minutes (9 states); the original W-method took 4 h 9 min (9 states) (the table row lacks an implementation label in the text I extracted).
- Result: "in three of the TLS implementations analysed new security flaws were found (in GnuTLS, the Java Secure Socket Extension, and OpenSSL)". "As our analysis of different TLS implementations resulted in different and unique state machines for each one, the technique can also be used for fingerprinting TLS implementations."
- GnuTLS: a handshake could complete with no integrity guarantee on earlier handshake messages; the client side showed the same; fixed in GnuTLS 3.3.9, and models of newer versions confirmed the fix.
- JSSE: after receiving ChangeCipherSpec the server answers a plaintext Finished with ChangeCipherSpec, sends encrypted data but expects plaintext from the client. Text: "This bug invalidates any assumption of integrity or confidentiality of data sent to the server, as it can be tricked into accepting plaintext data." Identified by CVE-2014-6593 (paper text near Section 4).
- OpenSSL: after an extra ChangeCipherSpec the server expects a ClientHello, responds, resets keys without using the new randoms, and "the same keys are used in both directions"; reported to the OpenSSL team.
- nqsb-TLS 0.4.0 (an OCaml TLS meant to be "both a specification and usable implementation"): learning revealed "alert messages are not encrypted even after a ChangeCipherSpec is received"; reported and fixed.
- Spec status: the paper states the TLS specifications are not completely unambiguous and that a state machine in the spec would remove room for interpretation. Beurdouche et al. (independent work) start from a manually constructed reference state machine instead of inferring one.
- Limits visible in the text: models cover the harness alphabet only; the paper compares implementations with each other because the specification has no state machine.

### A2. Model learning and model checking of SSH implementations (Fiterau-Brostean et al., SPIN 2017) [P] [FULL]
- URL: https://www.cs.ru.nl/E.Poll/papers/spin2017.pdf
- Bears on: D, H3 (non-Lean comparison), H4, H5.
- Read (full text obtained; the parts named here were read, other parts only skimmed): abstract, introduction, learning setup, property sections, results, conclusion.
- Method: model learning with abstraction (a mapper built from an existing SSH implementation) of OpenSSH, Bitvise and DropBear servers, then model checking (NuSMV) of properties formalised from the SSH RFCs.
- Conclusion, verbatim: "We have verified these properties on the learned models using model checking and have uncovered several minor standard violations. The security-critical properties were met by all implementations."
- Alphabets: a full alphabet for OpenSSH and a restricted one for Bitvise and DropBear; the authors say building the mapper "presented a considerable technical" challenge.
- Related work in the paper contrasts with static-analysis rule checking of SSH C code (Chen et al.; Udrea et al.), noting their rules were tied to code routines while these properties are abstract.
- Limits stated: bounded alphabet; the properties are the authors' formalisation of RFC text; the analysis considers protocol logic only.

### A3. Analysis of DTLS implementations using protocol state fuzzing (Fiterau-Brostean et al., USENIX Security 2020) [P] [FULL]
- URL: https://www.usenix.org/system/files/sec20-fiterau-brostean.pdf (talk page: https://www.usenix.org/conference/usenixsecurity20/presentation/fiterau-brostean)
- Bears on: D, H3 (non-Lean comparison), H4, H5.
- Read (full text obtained; the parts named here were read, other parts only skimmed): abstract, Sections 1 to 6 (setup, learning, findings per library), conclusion.
- Method: the authors extend TLS-Attacker with DTLS support suited to UDP, build a protocol-state-fuzzing framework for DTLS servers, and learn state machine models for thirteen DTLS implementations.
- Abstract, verbatim: "Analysis of the learned state models reveals four serious security vulnerabilities, including a full client authentication bypass in the latest JSSE version, as well as several functional bugs and non-conformance issues." (CVE-2020-2655 is the JSSE case.) Scandium is among the analysed libraries (it appears throughout the paper).
- Limits: models are over the harness alphabet; I did not record how the paper handles retransmission and timers.

### A4. State machine inference of QUIC (Rasool, Alpar et al., arXiv 1903.04384) [P] [FULL]
- URL: https://arxiv.org/pdf/1903.04384
- Bears on: D, H4, H5.
- Read (full text obtained; the parts named here were read, other parts only skimmed): abstract, Sections 1 to 6.
- Authors: Abdullah Rasool, Greg Alpar and others (first two named on the arXiv title page).
- Scope: a Google QUIC server learned with active learning and a mapper.
- Result, verbatim: "We do not see any unexpected states if we compare it with the available documentation." (said of two learned models, including one with 0-RTT). The paper also reports "some additional nondeterminism" while learning the 0-RTT model. No vulnerability is reported in the sections I read.
- Passive vs active: "Passive learning has a downside in that it depends on the states that previous executions have seen" is the authors' reason for active learning.

### A5. Model learning and model-based testing of MQTT brokers (Tappler, Aichernig, Bloem, arXiv 1904.07075) [P] [FULL]
- URL: https://arxiv.org/pdf/1904.07075
- Bears on: D, H3 (non-Lean comparison), H4, H5.
- Read (full text obtained; the parts named here were read, other parts only skimmed): abstract, learning and model-checking sections, results, conclusion.
- Scope: five open-source MQTT brokers learned, then compared pairwise by learned model and by model-based testing with counterexamples.
- Result, quoted from the conclusion: "In total we found 18 faults in four out of five brokers." Another passage states they "did not find any in the Mosquitto broker" and "found a part of the specification which strictly speaking none of the implementations" follows, i.e. a specification ambiguity.
- Limits stated: coarse bug patterns risk undetected bugs, so no filters were used; counterexample reduction would be needed for complex systems.

## Part B. Fuzzing guided by protocol state

### B1. AFLNet (Pham, Boehme, Roychoudhury, ICST 2020) [P] [FULL]
- URL: https://mboehme.github.io/paper/ICST20.AFLNet.pdf
- Bears on: D, H3 (non-Lean comparison), H4, H5.
- Read (full text obtained; the parts named here were read, other parts only skimmed): abstract, design, evaluation (RQs on coverage and vulnerabilities), threats.
- Design: state model inferred from server response codes; message sequences are the fuzzing unit; no protocol specification or grammar is required.
- Evaluation: baselines are BooFuzz (a stateful black-box fuzzer) and AFLnwe; 20 isolated instances per subject. Against BooFuzz, average branch, statement and state coverage increases are 60%, 56% and 67%. Against AFLnwe on LightFTP, 121%, 79% and 85%.
- Vulnerabilities, verbatim: "In total, AFLNet discovered four vulnerabilities in which two of them (CVE-2018-4013 and CVE-2019-7733) are known and the remaining two vulnerabilities (CVE-2019-7314 and CVE-2019-15232) are zero-day." Both zero-days scored CVSS 9.8.
- Limits: state coverage here means coverage of the inferred response-code state model, not of any specification.

### B2. StateAFL (Natella, Empirical Software Engineering 2022) [P] [FULL]
- URL: https://arxiv.org/pdf/2110.06253
- Bears on: D, H3 (non-Lean comparison), H4, H5.
- Read (full text obtained; the parts named here were read, other parts only skimmed): abstract, design (state inference from memory snapshots, fuzzy hashes), evaluation, threats to validity.
- Abstract, verbatim: the fuzzer "infers the current protocol state of the target server by taking snapshots of long-lived memory areas, and by applying a fuzzy hashing algorithm (Locality-Sensitive Hashing) to map memory contents to a unique state identifier." It works "with no manual customization" and "can achieve comparable, or even better code coverage and bug detection than customized fuzzing".
- Evaluation: integrated with a large set of network servers; numbers per target are in the PDF's tables (not transcribed here).
- Limits: state identity comes from memory contents, not from a specification; this is stated as a motivation, not a conformance property.

## Part C. Runtime verification and monitoring

### C1. SpecMon: Modular Black-Box Runtime Monitoring of Security Protocols (Morio, Kuennemann, CISPA; arXiv 2409.02918) [P] [FULL]
- URL: https://arxiv.org/pdf/2409.02918
- Bears on: D, H3 (non-Lean comparison), H4, H5.
- Read (full text obtained; the parts named here were read, other parts only skimmed): abstract, introduction, monitor construction, evaluation, limitations, related work.
- Method: turn a Tamarin multiset-rewrite model into a runtime monitor that runs against observed traffic and cryptographic library calls; black-box instrumentation of crypto and network libraries.
- Architecture text: the event stream contains "calls to cryptographic and networking functions including their inputs and computed outputs"; "If the event stream is a possible instantiation of a valid protocol run of the model, the program may continue, otherwise it is aborted." Online and offline modes. A WireGuard case appears in the paper.
- Results and overhead numbers are in the PDF's evaluation and are not transcribed here.
- Limits: the monitor sees events at the library boundary; the event aggregator must capture all events with matching function symbols in the model. The Dolev-Yao and trace-relative framing comes from the related Signal/WhatsApp paper (lead 15), not from this PDF.

### C2. Runtime protocol refinement checking (Ellsberg, Ding et al., NSDI 2025) [P] [FULL]
- URL: https://www.usenix.org/system/files/nsdi25-ding.pdf
- Bears on: D, H3 (non-Lean comparison), H4, H5.
- Read (full text obtained; the parts named here were read, other parts only skimmed): abstract, Sections 1 to 6 (definition of RPRC, implementation, evaluation, discussion).
- Abstract: "Ellsberg checks RPRC by comparing messages sent and received by each DPI process to those produced by a simulated execution of the protocol." Applied to etcd, ZooKeeper and Redis Raft; the abstract says it can "detect previously reported protocol bugs in these DPIs".
- Evaluation: a table lists previously reported bugs (for etcd, for example linearizable read, stale read after election, two reconfiguration bugs) that Ellsberg detects; the text also reports one previously unreported RedisRaft bug found while testing. I did not count a total "10" in the PDF text; an earlier draft of this file said 10 from a search snippet. That number is unverified.
- Limits quoted in the paper: Ellsberg "cannot prove that an implementation is correct because (a) it cannot detect refinement violations that do not result in the DPI sending out an inconsistent message; and (b) it is not guaranteed to have observed all possible DPI behaviors." It assumes the spec is correct and uses specifications written for sequential assumptions.

### C3. P and PObserve (AWS) [P] [FULL]
- URL: https://p-org.github.io/P/advanced/pobserve/pobserve/
- Bears on: D, H4, H5.
- Read (full text obtained; the parts named here were read, other parts only skimmed): the PObserve documentation page.
- Statement: PObserve checks service logs against P specifications; "as PObserve operates on service logs, it does not require additional instrumentation". Intended for post-hoc validation of distributed system logs, including logs from production.
- Limits: no coverage claim; checking depends on what events logs record.

## Part D. Lean and proof-assistant code as spec or oracle (repository inspection)

All repos cloned at depth 1 on 2026-11-07. AI-assistant files are noted as facts only.

| ID | Repo | SHA / date | Toolchain | Licence | Lean files / LOC | sorry | admit | axiom | native_decide | implemented_by | extern | opaque | partial | AI-assistant files |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| D1 | etheorem/etheorem | 417ca909 / 2026-11-01 | v4.34.1 | LGPL-3.0 | 369 / 61,202 | 0 | 0 | 5 | 662 | 10 | 39 | 41 | 10 | CLAUDE.md |
| D2 | c4milo/chapulin | 22a70c13 / 2026-11-06 | v4.33.0 (spec/lean) | Apache-2.0 | 60 / 29,486 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 1 | CLAUDE.md |
| D3 | pb64-lean/tls13-lean | 0c61f21f / 2026-09-09 | nightly-2026-04-25 | Apache-2.0 | 43 / 27,106 | 0 | 0 | 0 | 0 | 0 | 16 | 16 | 1 | none seen |
| D4 | AfonsoBitoque/LeanServer | 24b916aa / 2026-05-14 | v4.29.1 | Apache-2.0 | 108 / 38,581 | 0 | 0 | 5 | 399 | 0 | 77 | 77 | 5 | none seen |
| D5 | leanprover/LNSym | 49f8a1d9 / 2026-11-06 | v4.35.0 | Apache-2.0 | 202 / 45,232 | 38 | 0 | 1 | 194 | 1 | 0 | 0 | 53 | none seen |
| D6 | Verified-zkEVM/leanerVM | 25da57d3 / 2026-11-06 | v4.34.1 | Apache-2.0 | 425 / 118,403 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | AGENTS.md; commit co-authored by Claude |
| D7 | NyxFoundation/leanSSZ | e433fe11 / 2026-07-28 | v4.29.1 | none file found | 25 / 4,102 | 0 | 0 | 2 | 0 | 0 | 2 | 2 | 0 | none seen |
| D8 | athanor-ai/kairos-cedar | 82f34615 / 2026-05-16 | v4.29.1 (three sub-projects) | Apache-2.0 | 40 / 6,921 | 5 | 0 | 0 | 0 | 0 | 0 | 0 | 3 | none seen |
| D9 | jyh/x86lean | 1c4b1149 / 2026-11-06 | v4.32.0-rc1 | Apache-2.0 | 46 / 39,408 | 0 | 0 | 0 | 2 | 0 | 0 | 0 | 0 | CLAUDE.md |

Notes: the scan is a regular-expression pass with comments and strings stripped, not a Lean elaboration, so counts are approximate. The kairos-cedar clone omitted the cedar-full sub-project's larger files if over 1 MB (blob filter); the Soundness.lean file quoted below was present. NyxFoundation/verity (https://github.com/NyxFoundation/verity, SHA b669e5be, 2026-10-30, MIT) has no Lean files; it is a Rust runtime (CLAUDE.md and .claude present) whose README says it combines the runtime "with the formal-leanSpec Lean 4 model" and tests against "leanSpec fixtures"; the Lean model lives in another repo that I did not inspect.

AI-generation flags (rule 7): determined from files in the repo root (CLAUDE.md, AGENTS.md, .claude) and, for leanerVM, a `Co-Authored-By: Claude` trailer in the last commit. Absence of such files is not proof of human authorship.

### D1. Etheorem [CODE] (H2, H3, H5, D)
- URL: https://github.com/etheorem/etheorem (SHA 417ca909, see table).
- Read: README (layout, pyspec harness, verification status), docs pointers, every `axiom` declaration, counts.
- What it is: an executable Lean 4 version of the Ethereum consensus-layer spec. The Lean artifact is the implementation under test against the Python reference test vectors (pyspec consensus-spec-tests), so the conformance direction is Lean code checked against external vectors, not Lean spec checked against another implementation.
- Verification status as the README reports it: 19 items characterised and 58 touched out of 625 (my reading of its counts; see "Verification status" in the README). Conformance rests on test vectors; proofs are a minority.
- Axioms, verbatim, with paths:
  - packages/SizzLean/SizzLean/Hasher/Sha256Equiv.lean:101 `axiom sha256Hash_eq_spec : @LeanHazmat.Sha256.sha256Hash = LeanSha256.hash` (FFI hash equals the pure-Lean reference).
  - packages/SizzLean/SizzLean/Hasher/Sha256Equiv.lean:108 `axiom sha256Combine_eq_spec : @LeanHazmat.Sha256.sha256Combine = LeanSha256.combine`.
  - packages/SizzLean/SizzLean/Hasher/Sha256Batch.lean:61 `axiom sha256BatchCombine_eq_spec`.
  - packages/LeanPoseidonProofs/LeanPoseidonProofs/Primality.lean:43 `axiom bn254FrModulus_prime : Nat.Prime bn254FrModulus` and line 48 `axiom blsFrModulus_prime : Nat.Prime blsFrModulus`.
- Counts: 662 `native_decide`, 39 `extern`, 0 `sorry`.
- AI flag: CLAUDE.md in the repo root.
- Not done: I did not run the test suite.

### D2. chapulin [CODE] (H3, H4, H5, D)
- URL: https://github.com/c4milo/chapulin (SHA 22a70c13). Files read: README, docs/verification.md, spec/lean/CONTRACT.md, spec/lean/Spec/Handshake.lean (866 lines), test/handshake_sequence_test.c (header).
- Role: a C11 TLS 1.3 stack with an executable Lean 4 specification used as a differential oracle. CONTRACT.md: "Independence: write every function from the RFC text (cite the section in a doc comment). Never look at the C sources". docs/verification.md: "a differential oracle only works when a shared misreading cannot make both sides agree"; five named exceptions model the C code's own intermediate steps.
- Test volume reported for `make diff` (fixed seed; nightly runs use new seeds): 22,034 random-input comparisons; 1,095 QUIC rows; 1,501 x25519 comparisons; 4,427 wide P-256 comparisons; 6,722 PEM rows.
- Protocol-level oracle (verbatim, spec/lean/Spec/Handshake.lean header): "TLS 1.3 handshake message ordering per RFC 9846 §4, written from the RFC text as an executable oracle. The model tracks only the order of server-to-client messages after the ClientHello; each message body is assumed valid ... Fixed profile: at most one HelloRetryRequest round, no client certificate, and two auth modes". The state type has constructors `start | retried | gotSH | awaitCert | awaitCV | awaitFin | connected | closed`, and `def step (mode : Mode) : State → Msg → Option State` returns `none` when the client does not take the message.
- Test driver (test/handshake_sequence_test.c header, verbatim): "enumerate every server message sequence up to a depth, render each as real TLS records over a mock transport, run the real client, and require its accept/reject verdict to equal the Lean model's handshake_sequence oracle." The mock server derives real keys, so Finished MACs and record protection are genuine; "Only the pinned-mode signature check is stubbed".
- Stated domain limits (same header): "this oracle models raw-pin mode ... CA-mode builds ... no sequence oracle covers that arm — only the e2e run exercises CA-mode sequencing. This is an accepted, recorded limitation."
- Proof layer: CBMC harnesses cover 131 of 169 C sources within bounds; the doc has a section "What rests on tests, not proofs" and several harnesses recorded as returning no verdict within time limits. docs/verification.md: "CBMC found one real bug during development".
- Lean counts: 0 `sorry`, 0 `axiom`, 0 `native_decide`.
- AI flag: CLAUDE.md in repo root; single-maintainer community project; not peer reviewed.
- Not done: I did not build or run `make diff`.

### D3. tls13-lean [CODE] (H2, H3, D)
- URL: https://github.com/pb64-lean/tls13-lean (SHA 0c61f21f). Read: README, directory listing, declaration scan.
- Role: TLS 1.3 written in Lean with theorems over HACL* primitives (16 `extern`, 16 `opaque`); the Lean code is the implementation, not an oracle for another one. README mentions RFC 8448 vectors and interoperability runs with OpenSSL, curl and Go. Not run by me. 0 `sorry`, 0 `axiom`.

### D4. LeanServer [CODE] (H2, H3, D)
- URL: https://github.com/AfonsoBitoque/LeanServer (SHA 24b916aa). Read: README, THREAT_MODEL.md, the five `axiom` declarations, scan.
- THREAT_MODEL.md line 12: "LeanServer's 914 verified theorems (0 `sorry`, 0 `axiom`) provide the following guarantees".
- Source: five `axiom` declarations: LeanServer/Server/Concurrency.lean:395 `io_ref_atomic_ops`, :400 `io_ref_compound_not_atomic`; LeanServer/Spec/TLSSpec.lean:275 `axiom x25519_dh_commutativity : ∀ (a b : ByteArray), a.size = 32 → b.size = 32 → LeanServer.X25519.scalarmult a (LeanServer.X25519.scalarmult_base b) = LeanServer.X25519.scalarmult b (LeanServer.X25519.scalarmult_base a)`, :293 `ephemeral_key_independence`; LeanServer/Spec/ProtocolInvariants.lean:419 `gc_no_guaranteed_zeroization`. 399 `native_decide`, 77 `extern`.
- README total (935) differs from the THREAT_MODEL total (914) (see Conflicts).

### D5. LNSym [CODE] (H3, H5, D)
- URL: https://github.com/leanprover/LNSym (SHA 49f8a1d9). Read: README, Makefile, Arm/Cosim.lean header, scan.
- Role: Lean model of Armv8 instructions with proofs about AWS-LC style code. Makefile: `all: specs correctness proofs tests cosim`; `cosim:` runs `lake exe lnsym $(VERBOSE) --num-tests $(NUM_TESTS)`. README: the default `make` is `make all VERBOSE=--verbose NUM_TESTS=20`; "if you are not on an Aarch64 machine, conformance testing will be skipped".
- Cosim.lean builds random instruction streams (`Insts.rand : List (Cosim.CosimM (Option (BitVec 32)))`) for comparison with hardware execution.
- Scan: 38 `sorry` (mostly under Tests/Tactics and Proofs/SHA512), 1 `axiom` (Tactics/SkipProof.lean:39 `axiom skipProofAx {α : Sort _} : α`), 194 `native_decide`, 53 `partial def`.

### D6 to D9 (brief) [CODE]
- D6 leanerVM (https://github.com/Verified-zkEVM/leanerVM, SHA 25da57d3; H3, H5, D): README text separating differential agreement from proof: "Differential agreement is regression evidence; it is not presented as a proof about the Rust source". 425 Lean files, 0 `sorry`, 0 `axiom`. AI flag: AGENTS.md and Claude co-author trailer.
- D7 leanSSZ (https://github.com/NyxFoundation/leanSSZ, SHA e433fe11; H2): 2 axioms: LeanSSZ/Merkle/Root.lean:43 `axiom HasHTR.collisionResistance {T : Type} [HasHTR T] : ...` (commented as "a cryptographic assumption, not provable") and LeanSSZ/Hasher/Sha256FFI.lean:34 `axiom combineRaw_size : ∀ a b : ByteArray, (combineRaw a b).size = 32`.
- D8 kairos-cedar (https://github.com/athanor-ai/kairos-cedar, SHA 82f34615; H3, H5): README line 5: "Each generated `(Policy, Schema, Request)` tuple is well-typed by construction (sorry-free Lean soundness theorem) and is dispatched to three independent oracles: the Rust reference (`cedar-policy`), the Go reimplementation (`cedar-go`), ...". cedar-full/CedarFull/Soundness.lean header (line 35): "Total: 1 sorry, isolated to a single named blocker theorem." The file has 5 `sorry` occurrences in the clone and comments at lines 776 and 787 saying `extDecimalLit` and `extIpLit` are `sorry`. Community project, unreviewed; not built by me.
- D9 x86lean (https://github.com/jyh/x86lean, SHA 1c4b1149; H3, D): Lean x86 model; its README describes differential runs against the ACL2 x86isa model as oracle; I did not run them. CLAUDE.md present. 0 `sorry`, 0 `axiom`, 2 `native_decide`.

## Conflicts
1. Lean axioms in LeanServer: THREAT_MODEL.md line 12 says "0 `sorry`, 0 `axiom`"; the source has 5 `axiom` declarations (paths in D4). The same file counts 914 theorems and the README 935. Unresolved.
2. kairos-cedar sorry: README line 5 describes a "sorry-free Lean soundness theorem"; cedar-full/CedarFull/Soundness.lean says "Total: 1 sorry, isolated to a single named blocker theorem" and contains 5 `sorry` occurrences. The sub-projects may be different theorems; unresolved, not built.
3. Ellsberg bug count: earlier snippet said 10 bugs; the PDF text I read lists previously reported bugs plus one unreported bug and gives no total I could confirm.
4. ProFuzzBench size: the paper describes 10 protocols; the repository README lists 13 servers. Source: https://github.com/profuzzbench/profuzzbench (repo, not fully read). Unresolved.
5. Conformance direction in Lean projects differs: Etheorem is tested against external vectors (Lean as implementation under test); chapulin and Cedar use Lean as the oracle for another implementation. Different claims, both recorded.
6. Learned-model papers: A4 reports no unexpected QUIC behaviour while A1, A3, A5 report bugs in other protocols. No resolution needed; different targets.

## Gaps
Closed in this pass: Lean-repo inspection for nine repos with SHAs and verbatim axioms; automata-learning primary papers; spec-guided fuzzing numbers; Ellsberg bug count.
Remaining: eBay TLA+ trace validation source; Quint/Apalache MBT; Jepsen/Porcupine; Tamarin-to-code beyond SpecMon; Coq/Rocq QuickChick oracles; angr protocol-conformance paper; no Lean test suite was run (static inspection only); the Lean model behind verity was not inspected; Mocket, SandTable, LearnLib, NEZHA, HOL-TestGen, ioco not read in full (leads).

## Pointers (not re-read here)
- Cedar differential random testing: see research/references/ Cedar dossier. My first-pass notes (arXiv https://arxiv.org/html/2407.01688v1, extractor-based) are not used as evidence.
- Ivy QUIC testing (McMillan and Zuck): see research/references/ Ivy dossier.
- TLA+ trace validation, CCF and MongoDB (eXtreme Modelling): see research/references/ trace-validation dossiers.

## Unverified leads (excluded from evidence)
Each was seen only in search results or via an LLM-based page extractor, or only partly read. URL, [P] or [S].
1. Mocket (EuroSys 2023) [P]: https://wsdou.github.io/papers/2023-eurosys-mocket.pdf ; abstract read only (reports 7 bugs, 3 new, needs annotations).
2. iMocket [P]: https://dl.acm.org/doi/full/10.1145/3728883
3. SandTable [P]: https://dl.acm.org/doi/10.1145/3627703.3650077
4. Multi-grained specifications [P]: https://github.com/Disalg-ICS-NJU/Conformance
5. Model-guided fuzzing of distributed systems [P]: https://arxiv.org/html/2410.02307v1
6. Blockchain paper noting Mocket memory limits [P]: https://arxiv.org/html/2501.08550v2
7. LearnLib CAV 2015 [P]: https://link.springer.com/content/pdf/10.1007/978-3-319-21690-4_32.pdf ; LearnLib "10 years later" https://link.springer.com/10.1007/978-3-031-98685-7_7 ; site https://learnlib.de/
8. Vaandrager, Model learning (CACM) [P]: https://dl.acm.org/doi/abs/10.1145/2967606
9. Learn-Check-Test, eMRTD/UDS [P]: https://arxiv.org/html/2509.22215v2
10. PSF framework [P]: http://www.link.springer.com/chapter/10.1007/978-3-032-32519-8_9
11. ProFuzzBench [P]: https://github.com/profuzzbench/profuzzbench
12. Fuzzing surveys [S]: https://dl.acm.org/doi/10.1016/j.cose.2025.104683 ; https://link.springer.com/chapter/10.1007/978-981-92-6843-6_24
13. APFuzz [P]: https://ar5iv.labs.arxiv.org/html/2602.21892
14. Leucker and Schallhart, A Brief Account of Runtime Verification [P]: https://www.isp.uni-luebeck.de/sites/default/files/publications/jlap08_1.pdf (intro only; definition not captured in full)
15. SpecMon on Signal and WhatsApp [P]: https://arxiv.org/html/2609.11882
16. VeriFi [P]: https://arxiv.org/html/1808.03406v1 ; Scribble monitors https://kar.kent.ac.uk/43736/1/NBY14.pdf
17. NEZHA [P]: https://www.cs.columbia.edu/~suman/docs/nezha.pdf (abstract and tables skimmed)
18. Frankencerts via thesis [S]: https://repositum.tuwien.at/bitstream/20.500.12708/231110/4/Paracha-2026-Hallucinating%20Certificates%20Differential%20Testing%20of%20TLS%20Certi...-vor.pdf
19. RFCcert [P]: https://dl.acm.org/doi/10.1145/3180155.3180226
20. TLS-Anvil [P]: https://www.usenix.org/system/files/sec22-maehren.pdf ; Walz and Sikora https://dl.acm.org/doi/10.1109/TDSC.2017.2763947 ; SoK https://dl.acm.org/doi/abs/10.1145/3664476.3670871
21. Vanhoef, symbolic execution of security protocol implementations [P]: https://www.usenix.org/system/files/conference/woot18/woot18-paper-vanhoef.pdf (read through an extractor only)
22. Monitor-based testing with KLEE [P]: https://dl.acm.org/doi/10.1145/3664476.3664521
23. SymbexNet [P]: https://lsds.doc.ic.ac.uk/content/symbexnet-testing-network-protocol-implementations-symbolic-execution-and-rule-based
24. ISAKMP symbolic execution [P]: https://pmc.ncbi.nlm.nih.gov/articles/PMC5690638/
25. Symbolic execution survey [S]: https://arxiv.org/html/2508.06643 ; CycloneTCP https://ar5iv.labs.arxiv.org/html/2109.10347
26. KLEE and angr docs [P]: http://klee-se.org/releases/docs/v3.1/tutorials/using-symbolic/ ; https://docs.angr.io/en/latest/core-concepts/symbolic.html
27. Tretmans ioco [P]: https://www.cis.upenn.edu/~lee/05cis700/papers/BRT03.pdf ; uioco https://repository.tno.nl/SingleDoc?docId=55200
28. Microsoft Spec Explorer [P]: https://www.microsoft.com/en-us/research/project/model-based-testing-with-specexplorer/
29. HOL-TestGen [P]: https://logicalhacking.com/software/hol-testgen/ ; https://link.springer.com/10.1007/978-3-642-00593-0_28 ; firewall example https://logicalhacking.com/publications/brucker.ea-model-based-2008/brucker.ea-model-based-2008.pdf
30. Anthropic, formal verification with Rocq and fuzz tests [P, vendor]: https://www.anthropic.com/research/formal-verification (read via extractor)
31. Sail RISC-V and Lean extractions [P]: https://github.com/riscv/sail-riscv ; https://github.com/Verified-zkEVM/riscv-zkvm ; https://github.com/opencompl/sail-riscv-lean
32. EvmAsm [P]: https://reservoir.lean-lang.org/@Verified-zkEVM/EvmAsm/
33. NyxFoundation/verity [P]: https://github.com/NyxFoundation/verity
34. Etheorem forum post [S]: https://ethresear.ch/t/etheorem-update-the-complete-executable-consensus-specs-written-in-lean-4/26063
35. Amazon Science LNSym article [P, vendor]: https://www.amazon.science/blog/how-the-lean-language-brings-math-to-coding-and-coding-to-math ; CAV 2024 https://leodemoura.github.io/files/CAV2024.pdf
36. zero-to-qed model-checking chapter [S]: https://github.com/sdiehl/zero-to-qed/blob/main/docs/src/22_model_checking.md
37. Lean verification landscape [S]: https://github.com/Beneficial-AI-Foundation/probe-lean/blob/main/docs/lean-verification-landscape.md
38. ChatAFL (Meng et al., NDSS 2024) [P]: https://www.ndss-symposium.org/wp-content/uploads/2024-556-paper.pdf ; abstract and vulnerability table fragments only: nine bugs listed (seven Live555, one ProFTPD, one Kamailio) marked CVE-requested or fixed; headline 47.6% and 42.7% more state transitions than AFLNet and NSFuzz not re-confirmed.
39. Specula LLM trace validation [S, preprint, unusual ID]: https://ar5iv.labs.arxiv.org/html/2607.25333

## Open Questions
1. What is the exact total of bugs in Ellsberg's evaluation, and which were new?
2. Do any peer-reviewed Lean projects use Lean as a protocol-conformance oracle for stateful network code (beyond chapulin and Cedar's stateless engine)?
3. Can chapulin's handshake-model test and `make diff` be reproduced, and what does the Lean model of the handshake actually cover?
4. How much of the five LeanServer and five Etheorem axioms are covered by tests rather than proofs?
5. Which learned-model results (TLS, SSH, DTLS, MQTT) were checked against a formal RFC state machine, and which only compared implementations with each other?
6. What do spec-guided fuzzers report in terms of specification conformance rather than coverage of their own state model?
7. Which of the Lean projects in Part D have been reviewed or reproduced by someone other than their authors?
8. Does the Phase 1b dossier set cover the trace validation and Cedar entries I moved to Pointers, and are the numbers consistent with my first-pass notes?
