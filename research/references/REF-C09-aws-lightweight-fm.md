# REF-C09: AWS lightweight formal methods (ShardStore reference models, P and PObserve)

Kind: comparator (non-Lean). Access date for all sources: 2026-11-07.

## 1. Identity
| Field | Value |
|---|---|
| Subject (protocol or algorithm) | (a) Amazon S3 ShardStore storage node (LSM-tree index, chunk store, soft-updates crash consistency, concurrency), Rust. (b) P language design models for AWS distributed protocols (e.g. S3 strong consistency) and PObserve log conformance checking of implementations in "Rust or Java". |
| Spec it targets | No external standard. Executable reference models in Rust (ShardStore) [S1]; P state machines and global monitors [S2, S3]. |
| Organisation / authors | (a) Bornholt, Joshi, Astrauskas, Cully, Kragl, Markle, Sauri, Schleit, Slatton, Tasiran, Van Geffen, Warfield (AWS, UT Austin, ETH Zurich, UW) [S1]. (b) Brooker and Desai (AWS) [S2]; P team, `p-org/P` [S3]. |
| Primary URLs | https://cdn.amazon.science/77/5e/4a7c238f4ce890efdc325df83263/using-lightweight-formal-methods-to-validate-a-key-value-storage-node-in-amazon-s3-2.pdf ; https://cacm.acm.org/practice/systems-correctness-practices-at-amazon-web-services/ ; https://github.com/p-org/P |
| Repo, commit SHA, last commit date, licence | ShardStore source and reference models are not public [S1]. `awslabs/shuttle` `855ae5d92a02dd3c02ddd3ca99dfaeb3b5e31d5b`, 2026-11-05, Apache-2.0 [S4]. `p-org/P` `260da928f67569ca7a8c0039a02d0bad7cde6096`, 2026-08-03, MIT [S3]. |
| Lean toolchain (or other tool and version) | ShardStore: Rust `proptest`-style property-based testing, Loom and Shuttle stateless model checkers, Miri, Crux (bounded symbolic evaluation), early Prusti experiments [S1]. P: P compiler and checker (.NET), PObserve (Java) [S3]. |
| Maturity | Production. ShardStore checks are "release blockers for continuous delivery" [S1]; P used by teams for "S3, EBS ... DynamoDB, MemoryDB, Aurora ... EC2, IoT" [S2]. |

## 2. Claim as stated by the authors (verbatim)
- "We do not aim to achieve full formal verification, but instead emphasize automation, usability, and the ability to continually ensure correctness as both software and its specification evolve over time." [S1, abstract]
- "Our work has prevented 16 issues from reaching production, including subtle crash consistency and concurrency problems, and has been extended by non-formal-methods experts to check new features and properties as ShardStore has evolved." [S1, abstract]
- "We use property-based testing to check that implementation code refines the reference model: any observable behavior of the implementation must be allowed by the model." [S1 sec 4.1]
- "With PObserve, we take structured logs from the execution of distributed systems and validate post hoc that they match behaviors allowed by the formal P specification of the system." [S2]
- "PObserve is a distributed runtime monitoring framework that allows checking formal correctness specifications on systems implementation using service logs. As PObserve operates on service logs, it does not require any additional instrumentation" [S3, `Docs/docs/advanced/pobserve/pobserve.md`]

Use of words (sub-claim D): ShardStore uses "refines" and "formal methods" for randomized testing and bounded checking, while stating "our validation checks can miss bugs" [S1]. The CACM article separates these from proof: "This type of proof goes beyond testing. It is a proof in the mathematical sense." (about Cedar in Dafny) [S2]. P docs say "PObserve ensures that the **implementation conforms to the high-level design in deployment** — extending formal verification guarantees from design time into production." (`Docs/docs/whatisP.md:120`) [S5].

## 3. What is actually formalised
ShardStore [S1]:
- Reference models: "an executable specification in Rust that provides the same interface as the component but using a simpler implementation"; e.g. the LSM-tree index model "is a hash map". Reference models total 450 lines versus 44,048 implementation lines (Fig. 6).
- Durability property decomposed three ways: (1) sequential crash-free equivalence (same key-value mapping after each API call); (2) sequential crashing executions against a model extended with allowed losses, via `Dependency` objects: "persistence: if a dependency says an operation has persisted before a crash, it should be readable after a crash" and "forward progress: after a non-crashing shutdown, every operation's dependency should indicate it is persistent"; (3) concurrent crash-free executions checked for linearizability against sequential models.
- Harness example: `#[proptest] fn proptest_index(ops: Vec<IndexOp<u32, u32>>)` applying each op to `PersistentLSMTIndex` and `ReferenceIndex` and calling `compare_results!` then `check_invariants` (Fig. 3).
- Deserialisers: "for any sequence of on-disk bytes (up to a size bound), our deserializers cannot panic" (Crux) [S1 sec 7].
- Environment: in-memory user-space disk; fault model of fail-stop crashes (`DirtyReboot(RebootType)`), injected IO failures (`FailDiskOnce(ExtentId)`); resource exhaustion not tested [S1 sec 4.4].
P [S2, S3]:
- Designs modelled "as communicating state machines"; "Specifications in P are written as **global runtime monitors**. These monitors observe the execution of the system and can assert any global safety or liveness invariants. Monitors are synchronously composed with the P state machines." Each machine has "an **unbounded FIFO buffer**", and sends are "reliable, buffered, non-blocking, and directed" unless loss is modelled explicitly (P Semantics page) [S6].
- PObserve inputs: a service-specific log parser producing `PObserveEvent` (key, timestamp, event, logLine, customPayload), a sequencer that globally sorts by timestamp, and a demultiplexer feeding P monitors [S3 `logparser.md`, `pobserve.md`].

## 4. Property classes covered
| Class | Status | Evidence |
|---|---|---|
| Functional correctness | yes (tested) | Sequential equivalence to reference models under property-based testing [S1]. |
| Safety invariant | yes (tested / bounded) | Crash-consistency persistence, linearizability, read-after-write under Loom/Shuttle [S1]; P model checking of design invariants [S2]. |
| Liveness | partial | ShardStore "forward progress" after clean shutdown; deadlock detection by stateless model checking [S1]. P monitors may state liveness [S6]. |
| Refinement | partial (tested) | Authors frame property tests as refinement of the reference model; no proof [S1]. |
| Symbolic security | no | Not addressed. |
| Computational security | no | Not addressed (CACM mentions separate HOL Light RSA work, out of scope here) [S2]. |
| Parsing | partial | Panic-freedom of deserialisers proved up to a size bound with Crux; "parsing of S3's messaging protocol ... which we are still working on validating" [S1]. |
| Constant-time | no | Not addressed. |
| Undefined behaviour | partial | Test suite run under Miri (dynamic) [S1]. |

## 5. Link to implementation code
- ShardStore: same-language executable spec, differential property-based testing against the real implementation above an in-memory disk; reference models also used as unit-test mocks, which "helps keep the reference models up-to-date" [S1]. Concurrency: Loom (sound for bounded small harnesses, CDSChecker algorithm, release/acquire model) and Shuttle (randomized PCT, scales to end-to-end tests "over a million steps") [S1, S4]. Index category: differential-testing.
- P: design-level model checking is not linked to code. PObserve links by post-hoc runtime monitoring of logs (index category: trace-validation / runtime monitoring), with the parser and log content as the bridge [S2, S3].
- Guarantee: ShardStore: "their reporting success does not mean the code is correct, only that they could not find a bug" [S1 sec 8.3]. PObserve: per-log conformance to monitors; no published soundness statement found.

## 6. Trusted computing base and soundness hygiene
Lean-style counts not applicable. TCB of a pass:
- The reference model itself (450 lines), and its relaxed comparison after injected failures ("a simple 'has failed' flag that relaxes equivalence checks") [S1].
- Test generators and their argument biases; coverage metrics used to spot blind spots [S1 sec 4.2].
- Coarse crash-state generation ("Coarse flushes can miss bugs compared to existing techniques that exhaustively enumerate all crash states at the block level") [S1].
- In-memory disk replacing the real block device [S1].
- Loom memory model and bounds; Shuttle randomness (no completeness) [S1, S4].
- Crux size bound [S1].
- For PObserve: the per-service log parser, timestamp-based global ordering, and whether logs record all relevant events (log content is not checked against code) [S3].

## 7. Scale, effort, automation
- ShardStore code (Fig. 6): implementation 44,048; unit and integration tests 19,540; reference models 450; functional correctness checks 4,860; crash consistency checks 2,661; concurrency checks 901; total 72,460 lines of Rust [S1]. "These artifacts combined are only 13% of the total code base and 20% of the size of the implementation code, an overhead that compares favorably to formal verification approaches that report 3-10× overhead" [S1 sec 8.2]. Note the intro says the harnesses "comprise 12% of the ShardStore code base" (see Conflicts).
- Effort: "two formal methods experts working full-time for nine months and a third expert who joined for three months"; afterwards "18% of the lines of code in the test harnesses were last edited by a non-formal-methods expert" [S1].
- Scale of runs: "tens of millions of random test sequences before every ShardStore deployment" [S1].
- Bugs: 16 issues (5 functional, 5 crash consistency, 6 concurrency including one deadlock) [S1 Fig. 5]. Minimisation example: 61 operations with 9 crashes and 226 KiB reduced to 6 operations, 1 crash, 2 B [S1 sec 4.3].
- P: no public effort or bug counts for S3 strong consistency; CACM: "P helped eliminate several design-level bugs early" [S2]. Aurora commit cost cut "from 2 to 1.5 network roundtrips" via P and TLA+ modelling [S2].
- PObserve: `Src/PObserve` contains 13,714 lines of Java at the inspected commit [S3, wc -l].

## 8. Limitations stated by the authors (verbatim)
- "We do not currently check properties of concurrent crashing executions because we have not found an effective automated approach." [S1 sec 3.1]
- "these tests can only ever check system states that the test harness is able to reach." [S1 sec 4.2]
- "biasing introduces the risk of baking our assumptions into our tests" [S1]
- "We do not currently apply property-based testing for resource exhaustion failures ... because we lack a correctness oracle" [S1 sec 4.4]
- "sound stateless model checking tools like Loom are not scalable enough to check linearizability of end-to-end ShardStore tests" [S1 sec 6]
- "We have seen one example of a bug that our validation should have caught but did not, and was instead caught during manual code review." [S1 sec 8.3]
- "The primary barriers for formal methods tools include their steep learning curve and the specialized domain expertise required." [S2]

## 9. Independent evidence
- CCF authors contrast ShardStore: "Unlike the CCF approach however, state exploration is limited to what the test harness is able to reach." [REF-C08 S1 sec 9]
- The CACM article is by AWS authors and is a practice survey, not an independent evaluation [S2].
- PObserve origin: Desai's HPTS 2022 slides credit PObserve to "Nathan Taylor, Beyazit Yalcinkaya, Ankush Desai"; they say "P has enjoyed widespread use at AWS (S3, EBS, EC2, DBS, and IAM) to formally reason about system design and behavior. P specs are currently used only for design validation." and label the pipeline "Stream Processing functionality built using Apache Flink" [S7]. Current docs say Java Streams [S3].
- No public P model for S3 strong consistency was found; claims about it rest on AWS statements [S2].

## 10. Reproducibility
Not run: sandbox disk limit. ShardStore cannot be reproduced (closed source). Commands that would be used: P tutorial `cd /tmp/c09-p/Tutorial/1_ClientServer && p compile && p check -tc tcSingleClient -s 100` (from P docs, not executed); Shuttle `cd /tmp/c09-shuttle && cargo test` (not executed). Static inspection only: shallow clones, `wc -l`, README and docs read.

## 11. Generalisation
Pattern: write a small executable reference model in the implementation language, reuse it as a mock, and check conformance by randomized differential testing (sequential), extended with crash-state generation and stateless model checking (concurrency); separately, P design models plus log-based monitors.
Preconditions for an arbitrary codebase:
- Component APIs stable enough to model; "By focusing on modeling components whose APIs had become stable, we avoided too much churn" [S1 sec 8.4].
- Determinism controllable in tests (e.g. Rust `HashMap` randomisation had to be controlled for minimisation) [S1].
- Explicit persistence objects (`Dependency`) so crash specs can be stated per operation [S1].
- For PObserve: structured logs that expose spec-level events and timestamps sufficient for a global order [S3].
Language: Rust-specific tooling for ShardStore (Loom, Shuttle, Miri, Crux); the pattern itself is language-neutral.

## 12. Bearing on sub-claims (facts only, no verdict)
| Sub-claim | Fact from this reference | Source ID |
|---|---|---|
| H1 | Durability and crash consistency of a 44k-line storage node were expressed as executable Rust models (450 lines) plus Dependency-based persistence predicates. | S1 |
| H2 | No properties were proved about ShardStore code except bounded panic-freedom of deserialisers; Prusti proofs of models were "limited". | S1 |
| H3 | Same-language reference models linked to Rust code via property tests and stateless model checking; 16 issues prevented; success does not imply correctness. | S1 |
| H4 | 13-20% code overhead; formal-methods experts for 9 person-months x2 plus 3; then maintained largely by engineers without FM background. | S1 |
| H5 | A production team obtained implementation conformance evidence without proofs; AWS reserves proof for "critical security boundaries such as authorization and virtualization". | S1, S2 |
| D | "refines" and "formal methods" are used for testing; CACM uses "proof in the mathematical sense" only for Dafny/Kani/HOL Light work. | S1, S2 |

## 13. Conflicts
- ShardStore validation overhead: intro says "the combined property definitions and test harnesses comprise 12% of the ShardStore code base" [S1 sec 1]; sec 8.2 says "These artifacts combined are only 13% of the total code base and 20% of the size of the implementation code" [S1]. From Fig. 6: (450 + 4,860 + 2,661 + 901) / 72,460 is about 12.2%, and / 44,048 is about 20.1%. The 13% figure is not reproduced by Fig. 6.
- PObserve stream engine: HPTS 2022 slides say "Stream Processing functionality built using Apache Flink" [S7]; current docs say "The stream processing block in the figure is implemented using Java Streams" [S3]. Likely a change over time; not resolved.
- PObserve instrumentation: docs say it "does not require any additional instrumentation" [S3], while it does require a developer-written parser and structured logs containing the needed events [S3 `logparser.md`]. Both stated by the same source.

## 14. Sources
| ID | Title | URL | P/S | Depth | Summary |
|---|---|---|---|---|---|
| S1 | Bornholt et al., "Using Lightweight Formal Methods to Validate a Key-Value Storage Node in Amazon S3", SOSP 2021 | https://cdn.amazon.science/77/5e/4a7c238f4ce890efdc325df83263/using-lightweight-formal-methods-to-validate-a-key-value-storage-node-in-amazon-s3-2.pdf | P | FULL | Rust reference models, property-based conformance, crash and concurrency checking, 16 bugs, effort and LoC. |
| S2 | Brooker and Desai, "Systems Correctness Practices at Amazon Web Services", CACM 68(6) 2025 | https://cacm.acm.org/practice/systems-correctness-practices-at-amazon-web-services/ | P | FULL | AWS portfolio: P, PObserve, PBT, deterministic simulation, fuzzing, FIS, proofs (Cedar, Kani, HOL Light). |
| S3 | p-org/P repository incl. PObserve docs (commit 260da92) | https://github.com/p-org/P | P | CODE | P compiler/checker, PObserve source (13,714 Java lines), log parser and architecture docs. |
| S4 | awslabs/shuttle repository (commit 855ae5d) | https://github.com/awslabs/shuttle | P | CODE | Randomized concurrency testing library (PCT) used for ShardStore end-to-end tests. |
| S5 | P docs "What is P" (`Docs/docs/whatisP.md`, master) | https://github.com/p-org/P/blob/master/Docs/docs/whatisP.md | P | FULL | Checker backends (PSym, PEx, PVerifier); PObserve said to extend "formal verification guarantees" to production. |
| S6 | P docs "P Semantics" | https://p-org.github.io/P/advanced/psemantics/ | P | FULL | Actor-style machines, unbounded FIFO buffers, reliable sends, specs as global safety/liveness monitors. |
| S7 | Desai, "Formal Modeling and Analysis of Distributed Systems: Finding Critical Bugs Early!", HPTS 2022 slides (incl. PObserve poster by Taylor, Yalcinkaya, Desai) | https://hpts.ws/papers/2022/HPTS2022.pdf | P | FULL | 2022: P used only for design validation; PObserve pipeline on Apache Flink; JUnit/Log4J integration. |

## 15. Unverified leads (excluded from evidence)
- `p-org/PObserve` separate repository (https://github.com/p-org/PObserve) not inspected.
- Desai et al., "P: Safe asynchronous event-driven programming", PLDI 2013, and later P papers (ModP, PSym) not read.
- Cedar Dafny-plus-differential-testing (Amazon Science blog) is a stronger AWS example of spec-to-Rust conformance; not read here and may merit its own dossier.
- Reports that P specs are now generated with LLMs (PeasyAI in the P README; blog claims about Desai's LinkedIn posts) not verified.
- Christopher Meiklejohn CV claims foundational PObserve work "still used across AWS to detect bugs missed by testing" (search snippet only).
