# REF-C07: Ivy QUIC specification-based testing (McMillan and Zuck)

Kind: comparator (non-Lean). Access date for all sources: 2026-11-07.

## 1. Identity
| Field | Value |
|---|---|
| Subject (protocol or algorithm) | QUIC transport wire protocol (packet, frame, security-interface and application layers) |
| Spec it targets (e.g. RFC 8446 section) | IETF QUIC drafts, not RFC 9000. Paper: "track changes through nine drafts" ending at draft-18 [S1]; `microsoft/ivy` master README: "The currently targeted version is IETF draft 18" [S3]; Galois replication used a `quic23` branch (draft-23) [S4, S5]; Crochet et al. extended it to draft-29 [S6]. |
| Organisation / authors | Kenneth L. McMillan (Microsoft Research) and Lenore D. Zuck (University of Illinois Chicago) [S1, S2]. Replications: Galois (Matt Bauer, Mike Dodds) [S4]; Crochet, Rousseaux, Sambon, Piraux, Legay (EPIQ'21, UCLouvain authorship per paper header) [S6]. |
| Primary URLs | https://www.mcmil.net/pubs/SIGCOMM19.pdf ; https://www.mcmil.net/pubs/SECDEV19.pdf ; https://github.com/microsoft/ivy/tree/master/doc/examples/quic |
| Repo, commit SHA, last commit date, licence | `microsoft/ivy` master, commit `9f3c7ecc0b2383129fdd0953e10890d98d09a82d`, 2020-09-04, licence file `license.txt` ("IVy-tool", Microsoft licence text) [S3]. Galois demo `GaloisInc/ivy-quic-testing-demo` commit `d1571b79422b04df6985b069535e62afeca33300`, 2021-03-16, no licence file found [S5]. |
| Lean toolchain (or other tool and version) | Ivy (Python tool) compiling the spec to a C++ tester that calls the Z3 SMT solver; TLS 1.3 handshake supplied by a real TLS implementation (picotls in the Galois account) [S2, S4]. |
| Maturity | Research prototype. Galois: "this is definitely a research tool rather than something ready for deployment to customers" [S4]. Spec in master has not been updated since 2020 [S3]. |
| AI provenance | `none-found`: no agent configuration, co-author trailers or author statements on AI use were seen in the sources read or in the inspected Ivy QUIC files. |

## 2. Claim as stated by the authors (verbatim)
- "We develop a formal specification of the wire protocol, and use this specification to generate automated randomized testers for implementations of QUIC. The testers effectively take one role of the QUIC protocol, interacting with the other role to generate full protocol executions, and verifying that the implementations conform to the formal specification." [S1, abstract]
- "As a result, numerous implementation errors have been found. These include some vulnerabilities at the protocol and implementation levels, such as an off-path denial of service scenario and an information leak similar to the 'heartbleed' vulnerability in OpenSSL." [S1, abstract]
- "Compositional testing has the formal property that, if the system as a whole violates its specification, then some component must exhibit a failing test." [S1, sec 1.1]
- "Rule 1 provides a formal proof, but the premises of this proof are tested empirically." [S2, sec II]
- "An explicit non-goal of the study is to produce a full or complete specification of QUIC. The function of the specification in the case study is to aid in finding bugs in implementations of QUIC." [S2, sec VI]

Use of words (sub-claim D): the authors say the tester is "verifying that the implementations conform" [S1], but the mechanism is randomized testing against a monitor; the formal part is the soundness of the assume/guarantee rule (Theorems 1-3) whose premises are sampled, not proved [S2].

## 3. What is actually formalised
Two layers of formality.

(a) Theory (SECDEV'19, pen-and-paper, not mechanised) [S2]:
- A process is "a triple (I, O, T), where I ⊆ C is the input set, O ⊆ C is the output set and T is a prefix-closed set of (I ∪ O)-traces."
- Specifications are deterministic guarded-command acceptors: "A specification is a triple (Σ, s0, A) where Σ is a first-order state signature, the initial state s0 is a Σ-structure and A is a set of Σ-actions". Determinism is required "Because we need to use φ operationally as a checker".
- Theorem 1 (closed composition): from ⟨φ⟩ π1 ⟨φ⟩ and ⟨φ⟩ π2 ⟨φ⟩ infer π1 ∥ π2 ⊨ φ. Theorem 2 generalises to "locales" (overlapping subsets of processes, allowing unspecified "self-specified" components such as a third-party TLS library). Theorem 3: "⟨φ⟩ π ⟨φ⟩ holds iff M^φ_π ∥ π ⊨ φ", where the mirror M is the most general environment that does not break φ.
- Background assumptions β (e.g. nonce uniqueness) are added via rule (4): ⟨β ∧ φ⟩ ∥ℓ ⟨β ⇒ φ⟩ for all locales gives ∥Π ⊨ β ⇒ φ.
- An informal argument (explicitly "We will argue informally") that testing through a physical network is sound if the spec allows arbitrary delay and duplication.

(b) QUIC specification in Ivy (executable monitor) [S1, S3]. Protocol modelled as a single global, "process-oblivious" monitor over events at each layer. State is first-order functions and relations recording history; each event has a `before`/`around` guard (`require`) and an update. Size per SECDEV: "52 data type declarations, 45 state symbols ... and 30 actions" [S2]. Examples from the repo (commit 9f3c7ec):
- `doc/examples/quic/quic_packet.ivy:83` `action packet_event(src:ip.endpoint,dst:ip.endpoint,pkt:quic_packet) = {}`
- `quic_packet.ivy:268` `around packet_event(src:ip.endpoint,dst:ip.endpoint,pkt:quic_packet) {`
- `quic_packet.ivy:274` `require pkt.ptype ~= quic_packet_type.zero_rtt;` (0-RTT excluded)
- `quic_packet.ivy:321` `require pkt.seq_num <= last_pkt_num(scid,pkt.ptype) + 0x15;`
- `quic_packet.ivy:325` `require ~sent_pkt(scid,pkt.ptype,pkt.seq_num);  # [4]` (no packet-number reuse per encryption level)
- `quic_packet.ivy:330` `require num_queued_frames(scid) > 0;  # [7]`
- `quic_frame.ivy:556` `require f.data = stream_app_data(dcid,f.id).segment(offset,offset+f.length);` (STREAM bytes must equal application bytes at that offset; this is the check that caught the heartbleed-like leak)
- `quic_frame.ivy:568` `require ((offset) + (f.length)) <= stream_max_data(dcid,f.id);  # [2]` (flow control)

Environment and adversary: the network "provides no guarantees about delivery, except that it will not invent messages" [S2]. There is no explicit attacker model; the tester produces only legal (spec-conforming) stimulus, and "We are also interested in incorporating specific attacker models into the methodology" is listed as future work [S2]. Encryption/decryption, OS, network and TLS 1.3 are black boxes [S1, sec 4.4].

### 3a. Predicate semantics: what a passing test does and does not guarantee
There is no mechanised theorem. The paper theorems are pen-and-paper. Theorem 3 (SECDEV'19) states "⟨φ⟩ π ⟨φ⟩ holds iff M^φ_π ∥ π ⊨ φ", i.e. a process satisfies an assume/guarantee spec iff composing it with the mirror satisfies the spec [S2]. The tester runs one finite execution of that composition. The oracle at each wire event is the conjunction of Ivy `require` guards on the event, for example (commit `9f3c7ec`, `doc/examples/quic/`) [S3]:
- `quic_packet.ivy:325` `require ~sent_pkt(scid,pkt.ptype,pkt.seq_num);  # [4]`: a sender never reuses a packet number at one encryption level.
- `quic_frame.ivy:556` `require f.data = stream_app_data(dcid,f.id).segment(offset,offset+f.length);`: STREAM bytes equal the application data at that offset.
- `quic_frame.ivy:568` `require ((offset) + (f.length)) <= stream_max_data(dcid,f.id);  # [2]`: flow-control limit respected.

Plain language.
- A failing run shows a concrete execution where the implementation output violated a `require`, assuming the shim, the TLS library and the spec are correct. This is how the 27 errors were found [S1].
- A passing run shows only that the guards held on the events of that run. It does not show that the implementation satisfies the spec on all executions. The safety properties are therefore tested, not proved.
- Not covered: liveness, congestion control, error handling (in [S1]), serialisation formats, and any attacker behaviour beyond spec-legal stimulus [S1, S2, S4].

## 4. Property classes covered
| Class | Status | Evidence |
|---|---|---|
| Functional correctness | tested | Wire-level conformance of observable events only ("extensional"), tested not proved [S1 sec 4.1]. |
| Safety invariant | tested (not proved) | No protocol property is proved; the `require` guards are checked on each test run (see 3a). "Our formal specification captures only safety properties of the protocol." [S1 sec 6.1] |
| Liveness | no | "An important class of properties that we do not cover is liveness." Progress failures were recorded heuristically as "anomalously low transfer of data" [S1]. |
| Refinement | no | Spec "is not an abstract implementation of the protocol, but is instead an executable monitor" [S1]. Authors suggest internal statements might need "formal proof of refinement rather than testing" [S1 sec 6.2]. |
| Symbolic security | no | No adversary model; vulnerabilities found "as a side effect" [S1]. |
| Computational security | no | Crypto is a black box; "cryptographic randomness ... not testable externally" [S1]. |
| Parsing | no | Serialiser/deserialiser exist (`quic_ser.ivy`, `quic_deser.ivy`; README: "packet encoding and decoding are currently implemented in C++") but are part of the trusted shim, not specified [S3]. Galois: "does not specify serialization formats and SSL protocol logic" [S4]. |
| Constant-time | no | Not addressed. |
| Real time / congestion control | no | "we do not cover any statement in the QUIC document on recovery and congestion control" [S1]. |
| Error handling | no in [S1]; partial in [S6] | [S1]: "we do not capture protocol error handling requirements"; [S6] added transport-error-code tests by deliberately non-conforming generation. |

## 5. Link to implementation code
Mechanism: black-box, network-level specification-based testing (closest index category: differential-testing / runtime monitoring; not trace validation of internal state). The Ivy-generated tester (a "mirror") plays one role over real UDP; a shim written in Ivy serialises generated events to datagrams and deserialises received datagrams into `packet_event` calls; hidden frame/security/application events of the implementation are inferred from packets ("only 28 lines of Ivy code") [S1, S2]. Galois: "requires no source-code access or instrumentation" [S4]. Implementations tested: four servers in [S1] (one abandoned early for TLS 1.3 compatibility); test scripts list picoquic, quant and winquic [S3 `test/test.py:25-33`]; seven servers and seven clients in [S6].

Guarantee: none beyond sampled traces. The paper's "soundness" is probabilistic completeness of falsification: "if the sampling distribution is everywhere nonzero, there is a finite probably of producing the failure ... We do not, of course, test infinitely." [S2 sec III]. Runs were "limited to 100 protocol events and a single QUIC connection" [S1].

## 6. Trusted computing base and soundness hygiene
Lean-specific counts (sorry, axiom, native_decide etc.) are not applicable. TCB of a pass/fail verdict:
- The Ivy compiler and its C++ test generator, Z3 [S2].
- The shim, including packet encoding/decoding and QUIC packet protection ("This shim is complex because it includes packet encoding and decoding, that depend on QUIC's fairly elaborate encryption and decryption scheme") [S1 sec 4.5].
- The real TLS 1.3 library used as a "self-specified" component in the locale [S2].
- The informal network-soundness argument [S2 sec V-A].
- The specification itself, which is iteratively loosened and tightened against implementations; Galois note that classifying a failure as implementation bug versus spec too strong versus too weak "is a manual process of interpretation" [S4].
- Coverage-shaping constraints and weights added to the shim, which narrow the sampled space [S1 sec 6.3].
- Theorems 1-3 are paper proofs, not machine-checked [S2].

## 7. Scale, effort, automation
- Spec: 52 datatypes, 45 state symbols, 30 actions [S2]. Repo `doc/examples/quic/*.ivy`: 6,906 lines total (includes tests, serialisation, TLS glue); largest files `quic_frame.ivy` 1,297, `quic_client_test.ivy` 649, `quic_packet.ivy` 586 [S3, wc -l at 9f3c7ec].
- Effort: "the total amount of test engineer time we spent on actual testing was approximately four weeks"; spec effort across nine drafts described as "manageable" with low-level formatting and encryption the most time-consuming part [S1]. No figure for initial spec authoring.
- Generation rate: "approximately 10Hz (or < 0.01 Hz without dependent field extraction)" [S2]; generating "even a single, relatively simple, packet ... required several minutes" before the optimisation [S2 sec IV].
- Bug yield: 27 errors; 13 detected by spec violation, 12 by crash, 2 by lack of progress; 4 exploitable; 4 caused partly by ambiguities in the standard; 18 of 23 with known root cause due to adverse stimulus [S1 Table 1]. Typical rate "one" true error per engineer-day, maximum three [S1]. Rarest error (heartbleed-like leak) appeared "roughly every 20,000 events (about 40 minutes of testing)" [S1].
- Automation: fully automatic generation and checking; manual triage.

## 8. Limitations stated by the authors (verbatim)
- "We formally specified most features of the protocol, omitting 0-RTT data, version negotiation, retry messages, and one frame type (RETIRE_CONNECTION_ID)." [S1]
- "we found that a large fraction of the draft standard could not be formalized in a testable way. A frequent cause of this is that the statements refer to internal events within the implementation that have no clear definition or that cannot be inferred by observations on the wire." [S1]
- "We also found that a large portion (perhaps most) of the statements in the standard were not usefully testable." [S1 sec 6.2]
- "By their nature, such properties are difficult to test ... detecting liveness violations by testing is heuristic at best." [S1]
- "Sampling randomly from all legal inputs tends to produce poor coverage of implementation behaviors." [S1 sec 6.3]
- "the implementation behaviors cannot be fully covered without controlling their client API's and the underlying system API's." [S1]
- "It may be difficult in practice to determine why test generation is slow. A related problem is that test generation may deadlock because a requirement is unsatisfiable. ... We do not know if these methods will scale to more complex protocols that QUIC." [S1 sec 6.4]
- "a formal specification of a highly complex system need not and perhaps cannot be fully complete ... or even fully correct." [S2]
- "The method does not provide an approximation bound" (for the sampling distribution) [S2].

## 9. Independent evidence
- Standard change: off-path migration DoS: "Mitigations for this off-path migration attack were included in a later version of the standard." [S1]. Author-reported; the draft change itself was not independently checked here.
- Galois replication (2021) found the picoquic (draft-23 era version) server acknowledges ACK-only packets after adding that requirement to the spec themselves ("The Ivy QUIC specification did not specify this requirement, we so added it ourselves") [S4, S5 `resources/ack-patch.diff`]. Lessons: Quant failed with "TLS error 120"; output traces "difficult to interpret"; "Only someone deeply familiar with both the Ivy spec and the IETF spec can understand why a test has failed"; "PCAP trace capture does not work currently" [S4].
- Crochet et al. extended the spec to draft-29, tested seven servers and seven clients, reported per-test pass ratios and ambiguity examples (e.g. "highest-numbered non-probing packet" in draft-29 sec 9.3 admits two formal readings giving different results), and modified Ivy to support CIDs up to 16 bytes [S6]. Their spec was released under an anonymised repo (`AnonyQUIC/QUIC-Ivy`) [S6]; not inspected.
- Adoption: no evidence found that IETF adopted the Ivy spec as normative.

## 10. Reproducibility
Not built or run: sandbox disk limit. Ivy (Python 2-era tool), Z3, picotls and QUIC implementations would be needed. Command that would be used (from Galois demo README): `cd /tmp/c07-galois && ./run.sh` (builds a Docker image and runs 10 randomized tests of picoquic) [S5]. Static inspection only: both repos cloned with `git clone --depth 1`, files read, line counts taken.

## 11. Generalisation
Pattern: global executable monitor plus assume/guarantee mirror, used both as test generator and oracle, connected to an unmodified black-box implementation through a shim at an observable interface (here UDP).
To apply to an arbitrary codebase:
- An observable interface where all spec-relevant events can be captured or inferred (wire, API or log). Requirements about internal events must be dropped or need another method [S1 sec 6.2].
- A deterministic, guarded-command spec over first-order state, written so an SMT solver can sample guards quickly (dependent-field extraction, layered actions) [S2 sec IV].
- A shim per interface (serialisation, encryption), which becomes trusted.
- Concrete stand-ins for components lacking specs (locales) [S2].
Case-specific preconditions: QUIC's wire visibility after decryption by the tester; availability of picotls; many independent implementations from which to distil the spec; a fixed draft version matched to implementation versions [S4].
Language independence: the method is independent of the implementation language because it only touches the wire [S4]. This is the property relevant to H4.

## 12. Bearing on sub-claims (facts only, no verdict)
| Sub-claim | Fact from this reference | Source ID |
|---|---|---|
| H1 | A first-order guarded-command monitor expressed most of QUIC's wire protocol safety requirements (52 types, 45 state symbols, 30 actions) with unbounded connections, streams and data; liveness, real time and crypto were not expressed. | S1, S2 |
| H2 | No protocol property was proved; the compositional rule is proved on paper, premises are tested by sampling. | S2 |
| H3 | Spec linked to C/C++/Rust/Go/Python implementations by black-box network testing; guarantee is empirical (27 errors in 4 servers; more in 14 implementations in S6). | S1, S6 |
| H4 | Method needs only network access, no source or instrumentation; but needs a per-protocol shim and spec version matched to the implementation. | S4, S5 |
| H5 | Implementations were checked against a formal spec without any proof of protocol-level properties. | S1 |
| D | Authors use "verifying that the implementations conform" for randomized testing; Galois calls formal verification "the gold standard" and positions this as a cheaper alternative. | S1, S4 |

## 13. Conflicts
- Draft version: `microsoft/ivy` master README says "The currently targeted version is IETF draft 18" [S3]; Galois says "Ivy comes with a QUIC specification (based on IETF draft 23)" [S4], referring to a `quic23` branch of `kenmcmil/ivy` [S5].
- Number of implementations: SIGCOMM says four implementations tested, one abandoned early [S1]; SECDEV says "We applied the testing methodology to HTTP servers based on the four chosen implementations" [S2]. Consistent in count but S1 makes clear only three were tested substantively.
- Prior research topic file `research/05-implementation-conformance-methods.md` labels arXiv 2503.01374 as a "Galois ... draft-29 update". The PDF read in full is by Crochet, Rousseaux, Sambon, Piraux, Legay (EPIQ'21), not Galois [S6].
- Error classification: SIGCOMM Table 1 gives 13 spec-violation detections out of 27 [S1]; SECDEV gives percentages over 23 root-caused errors: "adverse stimulus (78%) or ... compliance violation (57%) or both" [S2]. 57% of 23 is about 13, consistent; recorded so readers do not mix denominators.

## 14. Sources
| ID | Title | URL | P/S | Depth | Summary |
|---|---|---|---|---|---|
| S1 | McMillan and Zuck, "Formal specification and testing of QUIC", SIGCOMM 2019 | https://www.mcmil.net/pubs/SIGCOMM19.pdf | P | [FULL] | Ivy wire spec of QUIC used as randomized tester; 27 errors in server implementations; safety only, 4 weeks testing. |
| S2 | McMillan and Zuck, "Compositional Testing of Internet Protocols", SecDev 2019 | https://www.mcmil.net/pubs/SECDEV19.pdf | P | [FULL] | Formal foundations: process model, global specs, Theorems 1-3, mirrors, SMT sampling with UNSAT cores, dependent field extraction. |
| S3 | microsoft/ivy repository, doc/examples/quic (commit 9f3c7ec) | https://github.com/microsoft/ivy/tree/master/doc/examples/quic | P | [CODE] | Ivy source of the QUIC spec, shim, test drivers; 6,906 lines of .ivy. |
| S4 | Bauer and Dodds, "QUIC Testing, a Quick Replication", Galois blog, 2021-03-22 | https://www.galois.com/articles/quic-testing-a-quick-replication | S | [FULL] | Independent replication on picoquic; found ACK-of-ACK non-conformance; usability limits. |
| S5 | GaloisInc/ivy-quic-testing-demo (commit d1571b7) | https://github.com/GaloisInc/ivy-quic-testing-demo | S | [CODE] | Docker demo, README and the patch adding the ACK-eliciting rule. |
| S6 | Crochet et al., "Verifying QUIC implementations using Ivy", EPIQ'21 (arXiv 2503.01374) | https://arxiv.org/pdf/2503.01374 | P | [FULL] | Extends Ivy QUIC spec to draft-29, tests 7 servers and 7 clients, adds error-code tests, documents spec ambiguities. |

## 15. Unverified leads (excluded from evidence)
- CACM "It takes a village: bridging the gaps between current and formal specifications for protocols" discusses this work: https://cacm.acm.org/research/it-takes-a-village-bridging-the-gaps-between-current-and-formal-specifications-for-protocols/ (not read).
- Later work by the Crochet/Legay group on Ivy-based QUIC/MinIP testing tools (e.g. "PANTHER" or attack-reproduction papers) was not located or read.
- `AnonyQUIC/QUIC-Ivy` repository with the draft-29 spec (not cloned).
- The exact IETF draft change mitigating off-path migration DoS (not traced to a draft diff).
