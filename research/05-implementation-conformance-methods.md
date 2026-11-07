# 05 Implementation Conformance Methods

Phase 1 evidence only. No analysis. Access date for every entry: 2026-11-07.
Tags: [P] primary (paper, repo, vendor or tool documentation by the authors), [S] secondary (blog, survey, summary, community post). Quotes come from fetched pages unless marked "snippet" (taken from a search result excerpt, not fully fetched).
Some arXiv IDs returned by the search index look unusual (26xx numbers). They are listed as indexed and not independently checked.

## 1. Model-based testing (MBT) and test generation from models

### 1.1 ioco theory and tools
- Tretmans, ioco: conformance is defined as "for all w in Straces(spec), out(imp after w) is a subset of out(spec after w)" (snippet). [P] https://www.cis.upenn.edu/~lee/05cis700/papers/BRT03.pdf (also https://scispace.com/pdf/model-based-testing-with-labelled-transition-systems-1ux2ml842s.pdf)
- Variants and critiques of ioco: uioco https://repository.tno.nl/SingleDoc?docId=55200 [P]; iocos https://link.springer.com/chapter/10.1007/978-3-642-38592-6_9 [P].
- Microsoft Spec Explorer, model-based testing from model programs. [P] https://www.microsoft.com/en-us/research/project/model-based-testing-with-specexplorer/ and https://learn.microsoft.com/es-es/previous-versions/visualstudio/spec-explorer/ee620411(v=specexplorer.10)

### 1.2 TLA+ model graphs to test cases
- MongoDB Realm Sync operational transformation (OT): TLC state graph (3 clients, 3-element array, one write each) gave 30,184 states and 4,913 behaviours, each turned into a C++ test. "These 4913 tests immediately achieved 100% branch coverage of the implementation, which we hadn't accomplished with our handwritten tests (21%) or millions of executions with the AFL fuzzer (92%)." [S, vendor blog] https://www.mongodb.com/company/blog/engineering/conformance-checking-at-mongodb-testing-our-code-matches-our-tla-specs
- Same case, paper: "MBTCG succeeded because the specification was deliberately written with a close correspondence to the implementation, via direct transcription." "The implementation under test was a self-contained function with defined inputs and output, of approximately one thousand lines of code." Effort: 4 weeks (2 for the TLA+ spec, 795 lines; 2 for the generator, 755 Go lines). "When MBTCG is possible, and when the state space is not too large, MBTCG is the gold standard of ensuring specification-implementation conformance." [P] Davis et al., eXtreme Modelling in Practice, VLDB 2020, https://ar5iv.arxiv.org/html/2006.00915
- Same paper, bug found: the TLA+ model checker hit a StackOverflowError caused by a non-terminating merge of ArraySwap and ArrayMove, "faithfully transcribed from C++ to TLA+". [P] same URL.
- MongoDB WiredTiger: path coverings of the TLC state graph converted into tens of thousands of test cases. [S, vendor blog] https://www.mongodb.com/company/blog/engineering/towards-model-based-verification-key-value-storage-engine ; related VLDB paper https://www.vldb.org/pvldb/vol18/p5045-schultz.pdf [P]; related blog https://www.mongodb.com/company/blog/engineering/formal-methods-beyond-correctness-isolation-permissiveness-distributed-transactions [S]; commentary http://muratbuffalo.blogspot.com/2025/05/modular-verification-of-mongodb.html [S].
- Mocket (EuroSys 2023): uses the TLC state graph to generate tests, forces a deterministic implementation to follow them, and compares runtime state to model state. [P] https://wsdou.github.io/papers/2023-eurosys-mocket.pdf ; repo https://github.com/tcse-iscas/Mocket ; extension iMocket https://dl.acm.org/doi/full/10.1145/3728883 [P].
- A blockchain paper reports Mocket ran out of memory (18 GB) on a 4-node, 4-round instance. [P] https://arxiv.org/html/2501.08550v2
- SandTable (EuroSys 2024): spec-level state exploration with a conformance-checking step. Reported 8 systems, 23 bugs (18 new), 114x to 2989x faster than implementation-level exploration. [P] https://dl.acm.org/doi/10.1145/3627703.3650077 ; repo https://github.com/tangruize/SandTable . Follow-up on multi-grained specifications (EuroSys 2025): https://github.com/Disalg-ICS-NJU/Conformance [P].
- Model-guided fuzzing of distributed systems: TLA+ model coverage guides schedule fuzzing. [P, preprint] https://arxiv.org/html/2410.02307v1

### 1.3 Theorem-prover-based test generation
- HOL-TestGen (Isabelle/HOL): test cases derived inside the prover, with explicit test hypotheses linking test and proof; generated scripts can test C implementations. [P] https://logicalhacking.com/software/hol-testgen/ ; https://link.springer.com/10.1007/978-3-642-00593-0_28 ; https://logicalhacking.com/research/theorem-prover-based-testing/
- HOL-TestGen examples: firewall conformance https://logicalhacking.com/publications/brucker.ea-model-based-2008/brucker.ea-model-based-2008.pdf [P]; IPC protocol testing https://usr.lmf.cnrs.fr/~wolff/papers/conf/2015-vstte-ipc-testing.pdf [P].

### 1.4 Protocol specs as test generators (Ivy)
- McMillan and Zuck, QUIC: network-centric compositional testing, specifications in Ivy, Z3 used for generation. Found an off-path DoS and a "heartbleed-like" information leak (snippet); requires no source code or instrumentation. [P] https://www.mcmil.net/pubs/SIGCOMM19.pdf ; https://www.mcmil.net/pubs/SECDEV19.pdf [P]; discussion https://cacm.acm.org/research/it-takes-a-village-bridging-the-gaps-between-current-and-formal-specifications-for-protocols/ [S].
- Galois replication of the Ivy QUIC testing: https://www.galois.com/articles/quic-testing-a-quick-replication [S]; draft-29 update https://arxiv.org/html/2503.01374 [P]; demo https://github.com/GaloisInc/ivy-quic-testing-demo [P].

## 2. Refinement checking of implementations

- Ellsberg (NSDI 2025): runtime protocol refinement checking. Compares messages sent and received by an implementation against a simulated protocol execution. Go implementation; applied to etcd, ZooKeeper, RedisRaft; 10 bugs reported. [P] https://www.usenix.org/conference/nsdi25/presentation/ding ; https://www.usenix.org/system/files/nsdi25-ding.pdf ; thesis https://cs.nyu.edu/media/publications/PhD_Thesis.pdf
- Ellsberg limitation, quoted: "Ellsberg cannot prove that an implementation is correct because (a) it cannot detect refinement violations that do not result in the DPI sending out an inconsistent message; and (b) it is not guaranteed to have observed all possible DPI behaviors." It assumes the specification is correct. [P] same PDF.
- Static refinement proofs (IronFleet, Verdi, seL4-style) are not covered here; see 04-other-tools-protocol-verification.md. They were not searched for this file.

## 3. Trace validation and trace checking

### 3.1 TLA+ trace validation
- Cirstea, Kuppe, Loillier, Merz (SEFM 2024): relate program traces to TLA+ specs by reducing the problem to constrained model checking with TLC. Java instrumentation API; traces hold only updates to selected spec variables. Quote: "detecting discrepancies between the specifications and the implementations in all cases" (snippet). [P] https://arxiv.org/html/2404.16075v1 ; https://dl.acm.org/doi/10.1007/978-3-031-77382-2_8 ; slides https://www.uni-muenster.de/IFIP-WG22/Web/meeting/Aachen25/merz-slides.pdf
- TLA+ documentation on trace validation: "TV is not exhaustive"; avoids symbolic interpretation of source code (snippet). [P] https://docs.tlapl.us/using:tlc:trace_validation ; manual https://github.com/tlaplus/vscode-tlaplus/blob/master/resources/knowledgebase/tlc-trace-validation.md
- Specula, LLM-agent-assisted trace validation (preprint, ID as indexed, unverified). [S] https://ar5iv.labs.arxiv.org/html/2607.25333
- eBay TLA+ trace validation: no source found (see Open Questions).

### 3.2 MongoDB (model-based trace checking, MBTC)
- Outcome: "We canceled the MBTC project because we had achieved disappointingly little utility from our efforts, and our revised estimate of the marginal cost made it clear that the project was not worthwhile." [P] https://ar5iv.arxiv.org/html/2006.00915
- Cited causes: "the system's enormous code size, ... its complex internal concurrency control, and ... the oversimplifications we had made when writing our TLA+ specifications, before we knew they would be used for MBTC." [P] same URL.
- Effort: two engineers, 10 weeks (event tracing 4 weeks, 570 C++ lines; spec update 3 weeks, 252 TLA+ lines; Python post-processor 3 weeks, 484 lines). Spec changes grew the state space "from 42,034 states to 371,368, and model-checking time from 2 seconds to 14 minutes." [P] same URL.
- Traces: the rollback fuzzer reproduced a known violation "only 4 steps from the trace's start and left the remaining 2,683 steps unchecked." [P] same URL.
- Counter-statement: "MBTC is also a good approach when the implementation's behavior is non-deterministic and cannot be forced to execute specific test sequences." [P] same URL. Blog version: "Both techniques can work, given the right circumstances and strategy. Both techniques can fail, too!" [S] https://www.mongodb.com/company/blog/engineering/conformance-checking-at-mongodb-testing-our-code-matches-our-tla-specs ; author summary https://emptysqua.re/blog/extreme-modelling-in-practice/ [S].

### 3.3 Microsoft CCF (Smart Casual Verification)
- Howard, Kuppe, Ashton, Chamayou, Crooks, NSDI 2025. TLA+ specs bound to the C++ implementation by trace validation, run in CI. [P] https://www.microsoft.com/en-us/research/wp-content/uploads/2024/07/nsdi25spring-final392.pdf ; session https://www.usenix.org/conference/nsdi25/technical-sessions ; repo https://github.com/microsoft/CCF
- Sizes (Table 1): consensus spec 1,134 lines, trace validation 369 lines, consensus implementation 2,174 lines; "the implementation is already 63 kLoC in C++." [P] same PDF.
- Result: "find six subtle bugs in the design and implementation before they could impact production" (five safety, one liveness). Individual bugs credit different techniques: 48 hours of exhaustive model checking on 128 cores; trace validation discovered the commit-advance-on-AE-NACK discrepancy; simulation found a 34-state counterexample. The paper gives no per-technique count. [P] same PDF.
- Definition: trace validation "verifies whether the set of behaviors T ... intersects with the set of behaviors S derived from the high-level spec, thereby checking that T ∩ S ≠ ∅." [P] same PDF.
- Difficulties quoted: "The granularity of some actions in the consensus spec did not align with the granularity of events in the traces." "Although our consensus specs explicitly modeled losing messages, message loss was not recorded in the trace." TLC was extended with DFS; consistency-spec validation went from about an hour (BFS) to under a second (DFS). [P] same PDF.
- Effort: "approximately two engineer-months, spread over four months" for the trace spec; about one engineer-week for the consistency spec. A fuzzing prototype for the consensus layer "was developed but ultimately abandoned since it failed to generate interesting behaviors". [P] same PDF.
- Secondary: Murat Demirbas on the paper, 17 actions, 13 variables http://muratbuffalo.blogspot.com/2025/02/smart-casual-verification-of.html [S]; Decentralized Thoughts https://decentralizedthoughts.github.io/2025-05-23-smart-casual-verification/ [S].

## 4. Fuzzing guided by protocol state machines

- AFLNet (ICST 2020): state inferred from server response codes. "No protocol specification or message grammars are required" (snippet). [P] https://mboehme.github.io/paper/ICST20.AFLNet.pdf ; https://github.com/aflnet/aflnet
- StateAFL (Natella, EMSE 2022): state inferred from fuzzy hashes of long-lived memory snapshots. [P] https://ar5iv.labs.arxiv.org/html/2110.06253 ; https://github.com/stateafl/stateafl
- ProFuzzBench (ISSTA 2021): benchmark of 13 network servers (TLS, SSH, DTLS, FTP and others). [P] https://github.com/profuzzbench/profuzzbench
- ChatAFL (NDSS 2024): an LLM reads RFCs to extract grammars and state. Reports 47.6% and 42.7% more state transitions than AFLNet and NSFuzz, and 9 previously unknown vulnerabilities. [P] https://www.ndss-symposium.org/ndss-paper/large-language-model-guided-protocol-fuzzing/
- Surveys and further work: https://dl.acm.org/doi/10.1016/j.cose.2025.104683 [S]; https://link.springer.com/chapter/10.1007/978-981-92-6843-6_24 [S]; APFuzz (ID as indexed) https://ar5iv.labs.arxiv.org/html/2602.21892 [P, preprint].

## 5. Active automata learning

- LearnLib: algorithms include L*, KV, TTT, L#, ADT. Equivalence queries are approximated by conformance testing (W-method, random walks). [P] https://learnlib.de/ ; https://github.com/LearnLib/learnlib ; CAV 2015 paper https://link.springer.com/content/pdf/10.1007/978-3-319-21690-4_32.pdf ; "10 years later" https://link.springer.com/10.1007/978-3-031-98685-7_7
- Vaandrager, "Model learning", CACM 2017. [P] https://dl.acm.org/doi/abs/10.1145/2967606
- TLS: de Ruiter and Poll (USENIX Security 2015). 9 implementations; new flaws in GnuTLS, JSSE and OpenSSL. Spurious behaviour "does require manual investigation" (snippet). [P] https://www.cs.ru.nl/~erikpoll/papers/tls_fuzzing.pdf . Poll's slide: "All implementations we analysed are different! Why doesn't the TLS spec include a state machine?" [S] https://www.cs.ru.nl/~erikpoll/talks/GDR2023.pdf
- SSH: Fiterau-Brostean et al., SPIN 2017. Learned models checked against security properties and RFC conformance; standard violations found. [P] https://www.cs.ru.nl/E.Poll/papers/spin2017.pdf
- DTLS: Fiterau-Brostean et al., USENIX Security 2020. 13 implementations; 4 serious vulnerabilities including the JSSE client-authentication bypass CVE-2020-2655; non-conformance issues. [P] https://www.usenix.org/conference/usenixsecurity20/presentation/fiterau-brostean
- QUIC: state machine inference of a Google QUIC server using StateLearner (LearnLib wrapper) plus a mapper. [P, preprint] https://arxiv.org/html/1903.04384 ; tooling https://github.com/QUICTester
- MQTT: Tappler, Aichernig, Bloem. 5 brokers learned and compared pairwise; 18 faults in 4 of the 5. [P] https://arxiv.org/html/1904.07075v1
- Other: Learn-Check-Test with LearnLib on eMRTD and UDS https://arxiv.org/html/2509.22215v2 [P, preprint]; PSF framework http://www.link.springer.com/chapter/10.1007/978-3-032-32519-8_9 [P].

## 6. Runtime verification and monitoring

- Definition: runtime verification is "checking whether a run of a system under scrutiny satisfies or violates a given correctness property". [P] Leucker and Schallhart, https://www.isp.uni-luebeck.de/sites/default/files/publications/jlap08_1.pdf
- AWS P and PObserve: logs from running services checked against P specifications. "validate post hoc that they match behaviors allowed by the formal P specification" (snippet). [P] https://p-org.github.io/P/advanced/pobserve/pobserve/ ; https://github.com/p-org/P ; [S] https://cacm.acm.org/practice/systems-correctness-practices-at-amazon-web-services/
- SpecMon: Tamarin multiset-rewrite model turned into a runtime monitor, black-box via crypto and network library instrumentation. [P, preprint] https://ar5iv.labs.arxiv.org/html/2409.02918 ; earlier work https://webspace.science.uu.nl/~4110161/pubs/morio-20.pdf [P]. Follow-up on Signal and WhatsApp (ID as indexed): https://arxiv.org/html/2609.11882 [P, preprint], which describes guarantees as "trace-relative" and inside the Dolev-Yao abstraction (snippet).
- Other monitors: VeriFi https://arxiv.org/html/1808.03406v1 [P]; Scribble session-type monitors https://kar.kent.ac.uk/43736/1/NBY14.pdf [P].
- Ellsberg (section 2) is also a runtime monitor.

## 7. Differential testing

- NEZHA (IEEE S&P 2017): 521 discrepancies found vs 10 and 19 for Frankencerts and Mucerts. [P] https://www.cs.columbia.edu/~suman/docs/nezha.pdf
- Frankencerts (Brubaker et al. 2014), described in a later thesis: points to "lack of ground truth about which TLS implementations are correct", with implementations acting as cross-referencing oracles (snippet). [S] https://repositum.tuwien.at/bitstream/20.500.12708/231110/4/Paracha-2026-Hallucinating%20Certificates%20Differential%20Testing%20of%20TLS%20Certi...-vor.pdf
- RFCcert (ICSE 2018): RFC rules plus dynamic symbolic execution. [P] https://dl.acm.org/doi/10.1145/3180155.3180226
- TLS handshake differential fuzzing (Walz and Sikora): https://dl.acm.org/doi/10.1109/TDSC.2017.2763947 [P]. TLS-Anvil: https://www.usenix.org/system/files/sec22-maehren.pdf [P]. SoK on TLS testing: https://dl.acm.org/doi/abs/10.1145/3664476.3670871 [S].
- Learned-model comparison across implementations: MQTT and TLS entries in section 5.
- Spec-vs-implementation differential testing with a formal model as one side: Cedar, section 9.

## 8. Symbolic execution against specs

- Vanhoef, WOOT 2018: KLEE extended; applied to three WPA2 4-way handshake clients (iwd, wpa_supplicant, MediaTek driver). Properties checked: authentication tags securely compared; decrypted data authenticated. Found timing-unsafe compares, an iwd DoS, a MediaTek stack overflow (CVE-2018-14525), a wpa_supplicant decryption oracle (CVE-2018-14526). Crypto is modelled under Dolev-Yao ("we assume cryptographic primitives are perfect and unbreakable"). Quoted limits: "certain queries to the SMT solver can be slow to practically unsolvable" and "a state explosion when analysing large programs"; the handshake code had to be isolated and driven by unit tests. [P] https://www.usenix.org/system/files/conference/woot18/woot18-paper-vanhoef.pdf
- Same paper on related work: "The downside of this approach [reference implementations in custom languages with proofs] is that existing protocol implementations cannot be verified in this manner." [P] same URL.
- Monitor-based testing with symbolic execution (KLEE; DTLS and QUIC; 22 unknown bugs, 21 fixed). [P] https://dl.acm.org/doi/10.1145/3664476.3664521
- SymbexNet (IEEE TSE 2014; KLEE with rule-based specs; Zeroconf, DHCP). [P] https://lsds.doc.ic.ac.uk/content/symbexnet-testing-network-protocol-implementations-symbolic-execution-and-rule-based
- Model-guided symbolic execution of ISAKMP. [P] https://pmc.ncbi.nlm.nih.gov/articles/PMC5690638/
- Survey tabulating spec-guided symbolic execution (Asadian et al. translate RFC rules into KLEE assertions). [S] https://arxiv.org/html/2508.06643
- CycloneTCP: SPARK plus KLEE checking TCP state transitions against RFC 793. [S, whitepaper] https://ar5iv.labs.arxiv.org/html/2109.10347
- Tool documentation only: KLEE http://klee-se.org/releases/docs/v3.1/tutorials/using-symbolic/ [P]; angr https://docs.angr.io/en/latest/core-concepts/symbolic.html [P]. No angr protocol-conformance paper was found.

## 9. Proof assistants and Lean as spec or test oracle

### 9.1 Cedar (AWS)
- Approach: Lean model proved correct; Rust production code checked by differential random testing (DRT). "we randomly generate millions of inputs—access requests, entities, and policies—and send them to both the Lean model and the corresponding Rust production implementation." "If the two versions agree on the output, then we obtain a piece of evidence that Rust production code is on par with the Lean model." [P] https://arxiv.org/html/2407.01688v1
- Bugs: proofs found 4 bugs in the validator; "DRT and PBT helped us find and fix 21 additional bugs". [P] same URL.
- Size: Lean model 1,673 lines, proofs 5,714 lines; Rust production 15,693 lines, tests 20,458. "The Lean models are an order of magnitude smaller than their Rust counterparts, which include optimizations, extra code to provide useful diagnostic output, and some unmodeled code." [P] same URL.
- Limits stated by the authors: "Unsurprisingly, Cedar's testing framework has missed bugs, too." The non-termination bug was missed "because the probability of generating inputs triggering this bug is extremely low. Our framework is also inherently unable to find bugs outside the scope of the test generators." Table 4 lists 9 bugs missed by DRT/PBT (evaluator `in` operation, invalid decimal literal, parser on malformed policies, some public APIs). "Complete line coverage alone does not guarantee effective testing." [P] same URL.
- Generation detail: type-directed generator (schema, then entities, then policies and requests); each target fuzzed for 6 hours daily on ECS. [P] same URL.
- Other Cedar sources: https://www.amazon.science/blog/how-we-built-cedar-with-automated-reasoning-and-differential-testing [P, vendor]; https://aws.amazon.com/blogs/opensource/lean-into-verified-software-development/ [P, vendor]; https://github.com/cedar-policy/cedar-spec [P]; https://docs.cedarpolicy.com/other/security.html [P]; https://lean-lang.org/use-cases/cedar/ [S, Lean site]. These report that the original model was Dafny and was moved to Lean, that a release does not ship unless "its model, proofs, and differential tests are up to date", and (lean-lang.org) that the Lean model runs at about 5 microseconds per test case vs 7 for Rust (snippets).
- Nightly volume of about 100M tests appears in secondary talks and posts, not in the paper: https://ai.engineer/talks/lRa9sPaMyy4-your-code-has-bugs-lean4-has-proofs [S]; https://williamzujkowski.github.io/posts/2026-07-30-prove-the-gate-not-the-agent/ [S].
- Survey statement: "Correspondence between Lean spec and Rust code is established via differential random testing, not formal proof." [S] https://github.com/Beneficial-AI-Foundation/probe-lean/blob/main/docs/lean-verification-landscape.md
- Community port: kairos-cedar, Lean-typed generator testing Rust and cedar-go. Quality unreviewed. [S] https://github.com/athanor-ai/kairos-cedar

### 9.2 Lean and hardware or ISA conformance
- LNSym (Armv8 in Lean): `make cosim` runs conformance testing against Aarch64 hardware, skipped on non-Aarch64 machines. [P] https://github.com/leanprover/LNSym ; Amazon Science: "Since Lean programs are executable, the specifications achieve a high degree of trust through thorough conformance testing" https://www.amazon.science/blog/how-the-lean-language-brings-math-to-coding-and-coding-to-math [P, vendor]; https://leodemoura.github.io/files/CAV2024.pdf [P]
- Sail RISC-V golden model, tested against DUTs with RISCOF; experimental Lean emulator. [P] https://github.com/riscv/sail-riscv . Lean extractions: https://github.com/Verified-zkEVM/riscv-zkvm [P]; https://github.com/opencompl/sail-riscv-lean (README: "neither executable nor polished", may be stale) [P]; overview https://github.com/Verified-zkEVM/Overview [P].
- x86lean: differential runs against the ACL2 x86isa oracle. [P, repo] https://github.com/jyh/x86lean
- xv6iris vtest: Lean machine model tested against QEMU. [P, repo] https://github.com/mit-pdos/xv6iris/blob/lean/tools/vtest/README.md
- EvmAsm README states "No conformance testing". [P, repo] https://reservoir.lean-lang.org/@Verified-zkEVM/EvmAsm/

### 9.3 Lean and protocol or consensus conformance
- Etheorem: executable Ethereum consensus spec in Lean 4, reported as passing pyspec consensus-spec-test vectors. [P, repo] https://github.com/etheorem/etheorem ; [S] https://ethresear.ch/t/etheorem-update-the-complete-executable-consensus-specs-written-in-lean-4/26063
- Sibling projects: leanSSZ https://github.com/NyxFoundation/leanSSZ [P]; Verity (Rust runtime with differential tests against leanSpec fixtures) https://github.com/NyxFoundation/verity [P]; leanerVM, README: "Differential agreement is regression evidence; it is not presented as a proof about the Rust source" https://github.com/Verified-zkEVM/leanerVM [P].
- chapulin (C11 TLS 1.3): `spec/lean/` holds an executable Lean 4 spec of crypto, key schedule and message formats; "`make diff` compares the C against it on random inputs from a fixed seed, and the nightly runs it on new seeds." "A test enumerates every server message sequence, up to a fixed length, that a Lean model of the handshake admits. It requires the real client's verdict on each to match the model's." CBMC harnesses are the proof layer; `docs/verification.md` lists "what rests on tests instead of proofs". Community project, not peer reviewed. [P, repo] https://github.com/c4milo/chapulin
- tls13-lean: Lean TLS 1.3 over HACL*, with theorems, RFC 8448 vectors and interop with OpenSSL, curl and Go. Community, unreviewed. [P, repo] https://github.com/pb64-lean/tls13-lean
- LeanServer: claims 935 theorems and a spec-model-implementation refinement. Not checked. [P, repo] https://github.com/AfonsoBitoque/LeanServer

### 9.4 Proof assistants as oracle or spec elsewhere
- Anthropic (Rocq, not Lean): fuzz tests turned into specs over compiled RISC-V code, then proved. "If Claude can prove this spec, the fuzz test on RISC-V becomes superfluous". Also an RFC 1950 spec with a proof about the Linux kernel zlib. Caveats quoted: "'Formally verified' does not mean 'bug-free.'" and "a proof checker can't tell you whether your spec matches reality." Lean is mentioned only for Mathlib. [P, vendor] https://www.anthropic.com/research/formal-verification
- zero-to-qed chapter: "Transferring a theorem requires semantic correspondence: testing supports confidence in that correspondence but does not generally prove it." [S] https://github.com/sdiehl/zero-to-qed/blob/main/docs/src/22_model_checking.md
- Agent-workflow skill page describing Lean as an additional oracle. Low weight. [S] https://skillsmp.com/de/creators/xaionaro/.config-claude/skills-proof-driven-development
- HOL-TestGen: see section 1.3.

## 10. Side-by-side summary of stated outcomes (no weighting)

| Method | Reported positive evidence | Reported negative or limiting evidence |
|---|---|---|
| TLA+ test generation | 100% branch coverage of a 1,000-line OT function (MongoDB); 23 bugs in 8 systems (SandTable) | Needs deterministic control and a small state space; Mocket out of memory at 18 GB on a 4-node case |
| TLA+ trace validation | Six bugs and CI use on 63 kLoC C++ (CCF) | MongoDB server effort cancelled after 10 weeks; "TV is not exhaustive"; granularity mismatches |
| Runtime refinement (Ellsberg) | 10 bugs in etcd, ZooKeeper, RedisRaft | Authors state it cannot prove correctness |
| State-machine fuzzing | 9 unknown vulnerabilities (ChatAFL); ProFuzzBench across 13 servers | Coverage-style guarantee only; inferred states are approximate |
| Automata learning | Flaws in TLS, SSH, DTLS, MQTT implementations | Manual investigation of models; equivalence queries are approximated by testing |
| Runtime verification | PObserve, SpecMon on real services | Guarantees are trace-relative |
| Differential testing | 521 discrepancies (NEZHA) | No ground truth when implementations disagree |
| Symbolic execution | 22 bugs (DTLS and QUIC monitors); WPA2 CVEs | SMT slowness, state explosion, manual harness isolation |
| Lean model plus DRT (Cedar) | 21 bugs from DRT/PBT plus 4 from proofs | Nine documented missed bugs; testing limited to generator scope |
| Lean model plus conformance tests (LNSym, Etheorem, chapulin) | Hardware cosimulation; test vector passes; model-admitted message sequences checked against a real client | Community projects mostly unreviewed; some README disclaimers |

## Open Questions

1. eBay's TLA+ trace validation experience was not located. Is there a public source?
2. Quint/Apalache model-based testing, Jepsen/Porcupine-style linearizability checkers and Tamarin/ProVerif-to-implementation tools beyond SpecMon were not searched.
3. No angr protocol-conformance primary source found; only tool documentation.
4. No Coq/Rocq QuickChick or similar proof-assistant oracle source found for protocol implementations.
5. Is there any peer-reviewed Lean-specific tool for protocol-implementation conformance (trace validation, state-machine-based test generation, refinement monitoring) beyond Cedar and LNSym? None found.
6. Which Lean-hosted projects (Etheorem, chapulin, tls13-lean, LeanServer) have independent review or reproducible results? Their claims here come from repos and posts.
7. Many quotes are from search snippets, not full fetches (marked "snippet"). Key papers (AFLNet, Ellsberg, de Ruiter and Poll, Fiterau-Brostean, NEZHA, McMillan and Zuck) need full-text quote checks.
8. Several preprints have unusual arXiv IDs (2602, 2607, 2609). Their existence and content need confirming.
9. How do Cedar's DRT bug-count and nightly-volume figures (paper vs talks) compare over time? The paper covers 2024; later numbers are secondary.
10. For what protocols (stateful, concurrent, networked) has Lean been used as the oracle side of differential or conformance testing? Cedar is a stateless authorization engine; chapulin and Etheorem are the only protocol-adjacent cases found.
