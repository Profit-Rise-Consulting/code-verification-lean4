# Conflict register (Phase 1 aggregate)

This register aggregates every recorded disagreement between sources in the Phase 1 corpus, for use in the Phase 2 council debate (docs/phase2/council-kit.md, rule 4: conflicts listed here must be argued, not ignored). It resolves nothing and gives no verdict on the hypothesis.

Sources aggregated: the Conflicts sections of research/01 to research/08; section 13 of every research/references/REF-*.md present at the time of writing (REF-C01 to REF-C11, REF-L01 to REF-L08, REF-L10 to REF-L23); and the "Unrecorded conflicts" table of research/REVIEW-phase1.md (every row is carried; the QUIC attribution row is entered as a correction with provenance). Later additions take the next free numbers.

Conventions:
- Entries are numbered CF-001 onward and are never renumbered. CF-001 to CF-119 follow document order; later additions take the next free number and are placed in their sub-claim section, so numbers after CF-119 are not in document order.
- Source IDs are written `file:ID`, for example `01:S19` (research/01) or `REF-L04:S2` (research/references/REF-L04-veil.md); a bare dossier ID such as `REF-L12 [CODE]` means the dossier's own repository inspection.
- Quotes are reproduced verbatim as recorded in the cited corpus file. Where a side is the researcher's own static count, the quoted text is the researcher's recorded statement, marked [CODE].
- Admissibility follows research/CONVENTIONS.md: a side counts only if it rests on [FULL], [SECT] or [CODE] sources. Conflicts with an excluded side are in the final section. Where only part of a recorded conflict rests on a lead, the admissible part is entered here and the lead part is listed in the final section.
- Deduplication: where several files record the same disagreement, one entry is made and the "Recorded in" line lists every recording.
- Status is "open" for every entry. Where a recording file or the review already gives a reading (for example "likely a version difference"), it is quoted, not adopted.

Entries per section: H1 Expressiveness: 11; H2 Provability at scale: 47; H3 Implementation link: 59; H4 Generality: 1; H5 Inference (H5a, H5b): 1; D Definitions: 4; Cross-cutting: 5; total 128.

## Index

| ID | Title | Sub-claims | Type | Files |
|---|---|---|---|---|
| CF-001 | Veil liveness support: papers and one README reading versus the current README | H1, H2 | doc-vs-doc | 01, 02, 06, 08 |
| CF-002 | Formal semantics of Lean IO and Task | H1 | definitional | 06 |
| CF-003 | Scope of vcgen versus shipped WP instances | H1, H4 | doc-vs-doc | 06 |
| CF-004 | Task blocking under threadpool starvation | H1, H3 | doc-vs-doc | 06 |
| CF-005 | Velvet's basis: built on Loom or not | H1, H2 | paper-vs-repo | REF-L06 |
| CF-006 | Loom master versus the Loom snapshot pinned by Veil | H1, H2 | interpretive | REF-L06 |
| CF-007 | Treatment of polynomial time in Lean cryptographic frameworks | H1, H2, D | definitional | 02 |
| CF-008 | Iris-Lean HeapLang: ported core or test material | H1, H2 | paper-vs-repo | 01 |
| CF-009 | lean-tla toolchain version | H1 | number mismatch | REF-L18 |
| CF-010 | Leslie sorry status | H2 | claim-vs-code | 01, REF-L19 |
| CF-011 | Leslie README example list versus repository | H2 | doc-vs-doc | REF-L19 |
| CF-012 | tla-plus-lean-workflow sorry count | H2, H3 | claim-vs-code | 01, 06, REF-L18 |
| CF-013 | Telltale axiom inventory | H2 | doc-vs-doc | 01, REF-L23, REVIEW-phase1 |
| CF-014 | Telltale placeholder families versus top-level verification claim | H2, D | claim-vs-code | 01, REF-L23, REVIEW-phase1 |
| CF-015 | leanbook Raft: conditional premises versus derived invariants | H2 | doc-vs-doc | 01 |
| CF-016 | raft-lean-squad: are the RaftReachable.step hypotheses discharged | H2, H3 | doc-vs-doc | 02, REF-L17, 08 |
| CF-017 | raft-lean-squad theorem and file counts | H2 | number mismatch | 01, REF-L17, 08 |
| CF-018 | raft-lean-squad toolchain version | H2 | number mismatch | 01, REF-L17, 08 |
| CF-019 | raft-lean-squad #guard test count | H2, H3 | number mismatch | 01, REF-L17 |
| CF-020 | raft-lean-squad blog "zero sorry" versus interim critique | H2 | doc-vs-doc | REF-L17 |
| CF-021 | Veil case-study count | H2 | number mismatch | 08, REF-L04 |
| CF-022 | Veil performance relative to Ivy | H2 | number mismatch | 06, REF-L04 |
| CF-023 | Veil proof-reconstruction cost | H2 | number mismatch | 08, 06 |
| CF-024 | Veil trust base: stated versus default | H2, D | claim-vs-code | REF-L04 |
| CF-025 | Velvet automation: SMT-based or grind-based | H2 | paper-vs-repo | REF-L06 |
| CF-026 | Velvet case-study count | H2 | number mismatch | REF-L06 |
| CF-027 | Scaling of verification condition generation | H2 | doc-vs-doc | 06 |
| CF-028 | Status and lineage of mvcgen | H2 | doc-vs-doc | 06 |
| CF-029 | LLM proof success rates for verified code | H2 | number mismatch | 06 |
| CF-030 | CatCrypt handshake-security claims versus availability | H2 | doc-vs-doc | 02 |
| CF-031 | Hydrozoan: machine-checked versus "Read critically" | H2, D | interpretive | 02 |
| CF-032 | IronFleet proof-to-code ratio | H2 | number mismatch | 04 |
| CF-033 | IronFleet liveness and correctness scope | H2, H3, D | claim-vs-code | REF-C02, 04, REVIEW-phase1, REF-C10 |
| CF-034 | IronKV line counts: Verus versus IronFleet | H2 | number mismatch | REF-C02 |
| CF-035 | Verdi Raft size | H2 | number mismatch | REF-C03 |
| CF-036 | Tamarin 5G-AKA model and lemma counts | H2 | paper-vs-repo | REF-C05 |
| CF-037 | TLS 1.3: Tamarin versus ProVerif findings and scope | H2 | doc-vs-doc | REF-C05, REF-C06 |
| CF-038 | WireGuard identity mis-binding | H2 | doc-vs-doc | REF-C06 |
| CF-039 | WireGuard: "formally verified secure network tunnel protocol" versus key-exchange-only scope | H2, D | interpretive | 07 |
| CF-040 | Noise Explorer pattern count | H2 | paper-vs-repo | REF-C06 |
| CF-041 | Tamarin soundness findings without maintainer response | H2, D | interpretive | 04 |
| CF-042 | Chronos-Vault theorem counts | H2 | number mismatch | 08 |
| CF-043 | Clean: Poseidon BN254 primality sorry | H2 | claim-vs-code | REF-L12 |
| CF-044 | SymCrypt kyber_terminates: proved or assumed | H2, H3 | doc-vs-doc | REF-L07 |
| CF-045 | SymCrypt NTT ring isomorphism and decryption-failure bounds | H2 | paper-vs-repo | REF-L07 |
| CF-046 | VCVio: lattice schemes "future work" versus present in repo | H2 | paper-vs-repo | REF-L10 |
| CF-047 | Hopscotch pins an older VCVio | H2 | number mismatch | REF-L10 |
| CF-048 | kairos-cedar: sorry-free versus one sorry | H2 | claim-vs-code | 05 |
| CF-049 | verified-consensus paper commit | H2 | doc-vs-doc | REF-L20 |
| CF-050 | Cedar forbid_trumps_permit statement | H2 | paper-vs-repo | REF-L01 |
| CF-051 | Cedar typecheck theorem conclusion versus README wording | H2, D | definitional | REVIEW-phase1, REF-L01 |
| CF-052 | Cedar SymCC proof status | H2 | paper-vs-repo | REF-L01 |
| CF-053 | SampCert proof size | H2 | number mismatch | REF-L02 |
| CF-054 | Leslie Rust verification files | H3 | claim-vs-code | 01, REF-L19 |
| CF-055 | tla-plus-lean-workflow: "Aeneas-verified" in abstract versus repo | H3 | paper-vs-repo | REF-L18 |
| CF-056 | Can Rust code be verified against a Lean model | H3, H4 | doc-vs-doc | 02 |
| CF-057 | LeanServer theorem and axiom counts | H2, H3 | claim-vs-code | 02, 05, REF-L14 |
| CF-058 | LeanServer "no spec-impl gap" versus FFI and threat model | H3 | claim-vs-code | 02, REF-L14 |
| CF-059 | quiche Lean theorem counts | H2, H3 | number mismatch | 02, 07, REF-L16 |
| CF-060 | quiche toolchain version | H3 | number mismatch | REF-L16 |
| CF-061 | VeriDNS "verified to the RFC" versus assumed oracle | H3, D | interpretive | 02 |
| CF-062 | VeriDNS TCP scope | H3 | claim-vs-code | 07, REF-L16 |
| CF-063 | VeriDNS ioResumeLoop_sound versus internal audit | H3 | doc-vs-doc | REF-L16 |
| CF-064 | LeanMqtt payload status | H3 | paper-vs-repo | 07, REF-L13 |
| CF-065 | LeanMqtt conformance statement count | H3 | number mismatch | REF-L13 |
| CF-066 | Lean Std.Http parser changes since snapshot | H3 | doc-vs-doc | REF-L14 |
| CF-067 | Native-evaluation compiler trust versus SampCert's extractor route | H3, D | interpretive | 03 |
| CF-068 | Aeneas and Verity "zero axioms/sorry" versus raw greps | H3 | claim-vs-code | 03 |
| CF-069 | hax Lean backend maturity | H3, H4 | doc-vs-doc | 03, REF-L08 |
| CF-070 | hax-lean and hax version lines | H3 | number mismatch | REF-L08 |
| CF-071 | hax and libcrux repository homes | H3 | doc-vs-doc | REF-L08 |
| CF-072 | hax `implies` semantics | H3 | doc-vs-doc | REF-L08 |
| CF-073 | Aeneas support for unsafe and union code | H3, H4 | doc-vs-doc | 08 |
| CF-074 | Aeneas supported loop and borrow subset | H3, H4 | doc-vs-doc | REF-L07 |
| CF-075 | Aeneas Result semantics across revisions (paper, pinned d71d2e3f, main) | H3 | paper-vs-repo | REF-L07, REVIEW-phase1 |
| CF-076 | SymCrypt paper Appendix C versus code | H3 | paper-vs-repo | REF-L07 |
| CF-077 | SymCrypt ML-KEM size | H3 | number mismatch | REF-L07 |
| CF-078 | Cedar Rust line counts | H3 | number mismatch | REF-L01 |
| CF-079 | Cedar Lean versus Rust execution speed | H3 | number mismatch | REF-L01 |
| CF-080 | Cedar bug counts | H3, H5b | number mismatch | REF-L01 |
| CF-081 | SampCert trusted C++ size | H3 | number mismatch | REF-L02 |
| CF-082 | LNSym proved listing versus ELF test listing | H3 | claim-vs-code | REF-L03 |
| CF-083 | pg-lean "no opaque" versus grep | H3 | claim-vs-code | REF-L11 |
| CF-084 | tls13-lean RFC 9846 versus RFC 8446 | H3 | doc-vs-doc | REF-L11 |
| CF-085 | Clean: "behave exactly as you described" versus trust boundaries | H3, D | doc-vs-doc | REF-L12 |
| CF-086 | miTLS verification against RFC 8446 | H3 | doc-vs-doc | REF-C01, 04 |
| CF-087 | miTLS deployment in Firefox NSS | H3 | doc-vs-doc | REF-C01 |
| CF-088 | HACL* size figures within the Everest retrospective | H3 | number mismatch | REF-C01 |
| CF-089 | Verdi "verified implementation" versus shim bugs | H3, D | doc-vs-doc | REF-C03 |
| CF-090 | s2n-tls TLS version scope | H3 | paper-vs-repo | REF-C04 |
| CF-091 | aws-lc verification scope | H3 | doc-vs-doc | REF-C04 |
| CF-092 | RefTLS: extracted model versus public repo | H3 | paper-vs-repo | REF-C06 |
| CF-093 | Noise Explorer generated-code size | H3 | paper-vs-repo | REF-C06 |
| CF-094 | Ivy QUIC targeted draft version | H3 | doc-vs-doc | REF-C07 |
| CF-095 | Ivy QUIC number of implementations tested | H3 | number mismatch | REF-C07 |
| CF-096 | Ivy QUIC error classification denominators | H3, H5b | number mismatch | REF-C07 |
| CF-097 | QUIC draft-29 paper attribution (correction with provenance) | H3 | doc-vs-doc | REF-C07, REVIEW-phase1 |
| CF-098 | MongoDB trace validation: new bug or known violation | H3, H5b | doc-vs-doc | REF-C08 |
| CF-099 | etcd trace validation bug yield | H3, H5b | doc-vs-doc | REF-C08 |
| CF-100 | CCF: TLA+ trace validation in CI versus Lean replay at HEAD | H3, H5b | paper-vs-repo | REF-C08 |
| CF-101 | CCF Lean: model inclusion aim versus replay limit | H3, H5b | doc-vs-doc | REF-L21 |
| CF-102 | CCF PR #8663 removals versus files at HEAD | H3 | claim-vs-code | REF-L21 |
| CF-103 | CCF Lean package name | H3 | doc-vs-doc | REF-L21 |
| CF-104 | CCF replay assurance-limits pointer | H3 | doc-vs-doc | REF-L21 |
| CF-105 | ShardStore validation overhead: 12% versus 13% | H3, H5b | number mismatch | REF-C09 |
| CF-106 | PObserve stream engine | H3 | doc-vs-doc | REF-C09 |
| CF-107 | PObserve instrumentation requirement | H3, H5b | doc-vs-doc | REF-C09 |
| CF-108 | Learned-model testing: no unexpected QUIC behaviour versus bugs elsewhere | H3, H5b | interpretive | 05 |
| CF-109 | Aeneas port onto SymM: in progress or future | H4 | doc-vs-doc | 08 |
| CF-110 | Conformance direction in Lean projects | H5b, D | definitional | 05 |
| CF-111 | "Verified": kernel-checked theorem versus differential-testing link | D, H3, H5b | definitional | 02, 03, 08 |
| CF-112 | Meaning of "refinement of an implementation" and "foundational" | D, H3 | definitional | 04 |
| CF-113 | Specification-based testing as verification (Ivy) | D, H5b | definitional | 04 |
| CF-114 | Specification fidelity: what Lean does not check | D, H3 | interpretive | 08 |
| CF-115 | Size of the Lean trusted base | H2, H3, D | interpretive | 06 |
| CF-116 | trustCompiler history | H3, D | doc-vs-doc | 03 |
| CF-117 | Evidence policy: meaning of [FULL] | D (affects admissibility for all sub-claims) | definitional | REVIEW-phase1 |
| CF-118 | Trace-validation and lightweight-FM claims resting partly on excerpts | H3, H5b | interpretive | REVIEW-phase1 |
| CF-119 | Catalogue yes/partial cells versus dossier qualifications | H2, H3, D | interpretive | REVIEW-phase1 |
| CF-120 | Perennial Lean branch versus Rocq branch: crash reasoning | H1, H2 | doc-vs-doc | REF-L22 |
| CF-121 | Telltale file, line and theorem totals | H2 | number mismatch | REF-L23, 01 |
| CF-122 | Perennial ArraySemantics soundness | H2, H3 | interpretive | REF-L22 |
| CF-123 | Perennial default branch | H3 | doc-vs-doc | REF-L22 |
| CF-124 | Wording of the CompCert guarantee | H3, D | doc-vs-doc | REF-C11, 03 |
| CF-125 | CakeML in-logic bootstrapping versus CompCert extraction trust | H3, D | interpretive | REF-C11 |
| CF-126 | Aneris network model: no duplication (paper) versus duplication step (code) | H1 | paper-vs-repo | REF-C10 |
| CF-127 | Aeneas and Charon soundness: trusted translators versus pen-and-paper metatheory with a repaired flaw | H3 | doc-vs-doc | REF-L07 |
| CF-128 | Lean-SMT finite-field reconstruction count (paper-internal): 247 versus 348 of 403 | H2 | number mismatch | 06, REVIEW-phase1 |

## H1 Expressiveness

### CF-001 Veil liveness support: papers and one README reading versus the current README
- Sub-claims: H1, H2
- Type: doc-vs-doc
- Side A: "Repo README: supports specifying, implementing, testing and proving safety; liveness is listed as future work." (02:D1, research/02-lean4-crypto-and-network-verification.md); the CAV 2025 paper "evaluates safety invariants only (no occurrence of \"liveness\" in the body)" (06:P01, research/06-lean4-language-limits-and-automation.md); "The papers E01/E03 evaluate safety/inductive invariants." (08:E01, 08:E03, research/08-academic-literature-and-community.md)
- Side B: "Veil is a framework for (1) specifying, (2) implementing, (3) model-checking, and (4) proving safety and liveness properties of state transition systems, with a focus on distributed protocols." (01:S19, README at SHA 57447722e7, research/01-lean4-protocol-modeling.md; same text at 06:P04 and REF-L04:S4, research/references/REF-L04-veil.md); HEAD "ships `Examples/Liveness` and `Examples/LivenessMC`" (08:C01)
- Why it matters for the debate: Whether Veil can state and prove liveness bears on the expressiveness and provability claims for temporal properties.
- Status: open
- Recorded in: 01 C3; 02 C-7; 06 C4; 08 C-1 (08 C-1 Side A also cites a snippet, L20, and 02 C-7 cites a partially read SoK; those parts are listed in the inadmissible section)

### CF-002 Formal semantics of Lean IO and Task
- Sub-claims: H1
- Type: definitional
- Side A: "no formal semantics of Lean's IO/Task exists at all [R]" (06:P09, leanin repository, research/06-lean4-language-limits-and-automation.md; the repository itself marks the claim `[R]`, an unverified recollection)
- Side B: "IO is a world-passing state monad; `Task` is a structure with a `get` field (V02, Init/Core.lean:624)." (06:E01, 06:V02, research/06-lean4-language-limits-and-automation.md; see also 01:S32)
- Why it matters for the debate: Whether a semantics of concurrency primitives exists affects whether the runtime behaviour of Lean protocol code can be specified.
- Status: open
- Recorded in: 06 C3

### CF-003 Scope of vcgen versus shipped WP instances
- Sub-claims: H1, H4
- Type: doc-vs-doc
- Side A: assertions "open vcgen to … probabilistic logics and separation logic" and programs can be "x64 assembler or WebAssembly" (06:V04, Lean 4.35 release notes, research/06-lean4-language-limits-and-automation.md)
- Side B: "the instance list in the checked-out source (V02) has WP instances for Id, StateT, ReaderT, ExceptT, OptionT, EStateM, Option, Except only, with no line naming IO/Task/Async in the searched directories" (06:V02, same file)
- Why it matters for the debate: The gap between framework generality and shipped instances bears on whether effectful protocol code can be expressed in the standard VC framework today.
- Status: open
- Recorded in: 06 C5

### CF-004 Task blocking under threadpool starvation
- Sub-claims: H1, H3
- Type: doc-vs-doc
- Side A: "nested `Task.get` inside `IO.asTask` deadlocks when tasks ≥ cores" (06:E09, Zulip 2023, research/06-lean4-language-limits-and-automation.md)
- Side B: "the maximum threadpool size is temporarily increased by one while waiting so as to ensure the process cannot be deadlocked by threadpool starvation" (06:E02 / 06:V02 docstring, same file)
- Why it matters for the debate: Runtime concurrency behaviour determines what a model of Lean-executed protocol code must capture.
- Status: open
- Recorded in: 06 C6

### CF-005 Velvet's basis: built on Loom or not
- Sub-claims: H1, H2
- Type: paper-vs-repo
- Side A: "Veil [27] is another multi-modal verifier built on top of Loom", with Velvet described as built on Loom (REF-L06:S1, REF-L06:S3, research/references/REF-L06-loom-velvet.md)
- Side B: "Velvet `main` at `b2a349a` has no Loom dependency and uses a modified copy of Lean core's `vcgen` with `grind`." (REF-L06:S4, same file)
- Why it matters for the debate: Which framework underlies a verifier changes what its published foundational guarantees apply to.
- Status: open
- Recorded in: REF-L06 §13

### CF-006 Loom master versus the Loom snapshot pinned by Veil
- Sub-claims: H1, H2
- Type: interpretive
- Side A: "master (v4.24, Mathlib, `axiom trust_smt`)" (REF-L06:S2 @ 78928ab, research/references/REF-L06-loom-velvet.md)
- Side B: "the snapshot Veil pins (v4.35, no Mathlib, no SMT module, no axiom declaration)" (REF-L06:S2 @ f0ea910, same file)
- Why it matters for the debate: Statements about "Loom" may refer to materially different code bases and trust bases.
- Status: open
- Recorded in: REF-L06 §13

### CF-007 Treatment of polynomial time in Lean cryptographic frameworks
- Sub-claims: H1, H2, D
- Type: definitional
- Side A: "Since Lean currently misses a library for reasoning about the running time of algorithms, we use an axiomatic approach to model polynomial time." (02:V4, ePrint 2025/1700, research/02-lean4-crypto-and-network-verification.md)
- Side B: "HOPSCOTCH does not verify the polynomial-time complexity of the reductions used in the proof. Instead, we rely on the concrete security bounds, or on verifying this external condition by the user." (02:V3); CatCrypt "states concrete (not asymptotic) bounds" (02:C1); same file
- Why it matters for the debate: Different treatments of efficiency change what a "computational security" proof in Lean establishes.
- Status: open
- Recorded in: 02 C-3

### CF-008 Iris-Lean HeapLang: ported core or test material
- Sub-claims: H1, H2
- Type: paper-vs-repo
- Side A: the authors "ported more than 80% of the separation logic for the HeapLang programming language, including a variety of examples" (01:S30, Iris in Lean paper, research/01-lean4-protocol-modeling.md)
- Side B: "in the repository HeapLang sits in `Iris/IrisTest/HeapLang` (a test-named directory) and the only `axiom` declarations are in a benchmark file" (01:S29, same file)
- Why it matters for the debate: Whether a separation logic for a programming language is a maintained part of Iris-Lean affects claims about expressing heap-manipulating implementations.
- Status: open
- Recorded in: 01 C8

### CF-009 lean-tla toolchain version
- Sub-claims: H1
- Type: number mismatch
- Side A: "lean-tla README says toolchain `v4.33.0-rc1`" (REF-L18:S4, research/references/REF-L18-leanda-tla-lean.md)
- Side B: "`lean-toolchain` file and AGENTS.md say `v4.33.0-rc2`" (REF-L18:S4, same file)
- Why it matters for the debate: Reproducibility depends on which toolchain checked the proofs.
- Status: open
- Recorded in: REF-L18 §13

### CF-120 Perennial Lean branch versus Rocq branch: crash reasoning
- Sub-claims: H1, H2
- Type: doc-vs-doc
- Side A: the Rocq master README describes "crash weakest preconditions, crash invariants, idempotence, and crash refinement reasoning" in `src/program_logic/` (REF-L22:S5, research/references/REF-L22-perennial-goose.md)
- Side B: the Lean README says the Lean branch has "no crash program logic" and no `wpc` (REF-L22:S1); AGENTS.md calls the Rocq sources "frozen and out of date" and says "the Lean statements are authoritative" (REF-L22:S6)
- Why it matters for the debate: Claims about Perennial's crash-reasoning capability in Lean depend on which branch is meant.
- Status: open
- Recorded in: REF-L22 §13

### CF-126 Aneris network model: no duplication (paper) versus duplication step (code)
- Sub-claims: H1
- Type: paper-vs-repo
- Side A: the ESOP 2020 Aneris paper "assumed a network that does not duplicate" (REF-C10:S4, research/references/REF-C10-aneris-trillium-grove-fairneris.md; dossier paraphrase, no verbatim sentence from the paper is recorded)
- Side B: "`aneris/aneris_lang/lang.v:1032` `MessageDuplicateStep` adds a copy of an in-flight message (`state_ms σ ⊎ {[+ m +]}`)" (REF-C10:S5, logsem/aneris @ 59ae379)
- Why it matters for the debate: Statements about which network faults Aneris-based proofs cover depend on whether the 2020 paper or the current code is cited.
- Status: open
- Recorded in: REF-C10 §13

## H2 Provability at scale

### CF-010 Leslie sorry status
- Sub-claims: H2
- Type: claim-vs-code
- Side A: README table: "Paxos ... Spec complete, 2 sorry", "VRViewChange ... 1 sorry"; `PaxosRefinement.lean` docstring: "abstract-ahead, 1 sorry"; `PaxosEvaluation.md`: "All proofs are machine-checked and sorry-free." (01:S4, research/01-lean4-protocol-modeling.md; REF-L19:S1, research/references/REF-L19-leslie.md)
- Side B: "Code scan at SHA 58e58b2502: 5 `sorry` tokens in total, in `Refinement.lean` (2), `Examples/LastVoting.lean` (2) and `Prob/Spike/ASTSanity.lean` (1); none in Paxos files" (01:S4 [CODE]); "Code at `58e58b2`: 0 `sorry` in `Paxos.lean` and `VRViewChange.lean`." (REF-L19)
- Why it matters for the debate: Completeness of Leslie's consensus proofs depends on which of four statements holds at the cited commit.
- Status: open
- Recorded in: 01 C1; REF-L19 §13

### CF-011 Leslie README example list versus repository
- Sub-claims: H2
- Type: doc-vs-doc
- Side A: "README Project Structure lists 11 examples." (REF-L19:S1, research/references/REF-L19-leslie.md)
- Side B: "`Leslie/Examples` has more than 40 files (e.g. BenOr, LastVoting, ByzantineReliableBroadcast, FencingTokens)." (REF-L19:S1 [CODE], same file)
- Why it matters for the debate: The number of protocols covered by Leslie differs depending on which listing is used.
- Status: open
- Recorded in: REF-L19 §13

### CF-012 tla-plus-lean-workflow sorry count
- Sub-claims: H2, H3
- Type: claim-vs-code
- Side A: "The single remaining sorry stub" (06:P07 README, research/06-lean4-language-limits-and-automation.md); source comments say proof obligations are "marked `sorry` pending Veil/SMT discharge" and mention "two sorry stubs" (01:S27, research/01-lean4-protocol-modeling.md); OpenRaft header: "All proof obligations are marked `sorry` pending Veil/SMT discharge." (REF-L18:S5, research/references/REF-L18-leanda-tla-lean.md)
- Side B: "The token scan finds 0 `sorry`" (01:S27); "Code at `9910342`: 0 `sorry` tokens outside comments and strings." (REF-L18:S5); "the table count finds 8 occurrences of `sorry` (unclassified; may be comments)" (06:P07)
- Why it matters for the debate: Whether the Raft election safety proof in this artifact is complete depends on reconciling prose, comments and code.
- Status: open
- Recorded in: 01 C6; 06 C10; REF-L18 §13

### CF-013 Telltale axiom inventory
- Sub-claims: H2
- Type: doc-vs-doc
- Side A: `lean/CODE_MAP.md`: "The repository is currently axiom-free"; the Telltale scan "has 0 axioms in 830 files" (01:S15, research/01-lean4-protocol-modeling.md)
- Side B: "the same file's table still lists files with \"2 axioms\" and \"8 axioms\"" (01:S15); the same author's rumpsteak-aura "has 35 axioms in 221 files, including `deadlock_free` and `not_stuck`" (01:S16); at c8c6a7f, "CODE_MAP per-file rows say Monitor/Preservation has \"2 axioms\" and Deployment/Linking \"8 axioms\" (`CODE_MAP.md:560,568`), while the same file's global inventory (`:1024`, `:1035`) and a lexical check of the current files find 0 axioms" (REF-L23:S8, research/references/REF-L23-telltale.md)
- Why it matters for the debate: An axiom-free headline and residual axiom counts give different trust bases for the same proof stack.
- Status: open
- Recorded in: 01 C7; REF-L23 §13; REVIEW-phase1 "Unrecorded conflicts" (Telltale row)

### CF-014 Telltale placeholder families versus top-level verification claim
- Sub-claims: H2, D
- Type: claim-vs-code
- Side A: "formally verified for the declared shipped surface" (01:S15, Telltale top-level claim, research/01-lean4-protocol-modeling.md)
- Side B: the distributed-family table "marks Nakamoto and Reconfiguration as `placeholder` (\"some exported guarantee is vacuous or backed by True-style assumptions\")" (01:S15, same file). Within Side B the dossier records a further difference of degree: "`lean/Distributed/README.md:34` labels Nakamoto \"placeholder\" while `Nakamoto/Core.lean:208` proves common prefix from a longest-chain refinement assumption plus `LongestChain.common_prefix`; the probability budget remains an assumption" (REF-L23:S6, research/references/REF-L23-telltale.md)
- Why it matters for the debate: Quoting only the headline would conceal vacuous guarantees inside the claimed verified surface (REVIEW-phase1 asks that this not be erased).
- Status: open
- Recorded in: 01 C7; REF-L23 §13; REVIEW-phase1 "Unrecorded conflicts" (Telltale row)

### CF-015 leanbook Raft: conditional premises versus derived invariants
- Sub-claims: H2
- Type: doc-vs-doc
- Side A: "Conditional Raft Leader Completeness" lists Log Matching and election restrictions as "explicit premises ... not proved invariants of network..." (01:S26, research/01-lean4-protocol-modeling.md)
- Side B: the repo's summary says its network-induction module "derives global Log Matching" and a complete-bridge module "derives Leader Completeness" (01:S26, same file)
- Why it matters for the debate: Whether Raft safety is proved from the network model or assumed changes its weight as provability evidence.
- Status: open
- Recorded in: 01 C9

### CF-016 raft-lean-squad: are the RaftReachable.step hypotheses discharged
- Sub-claims: H2, H3
- Type: doc-vs-doc
- Side A: REPORT.md: "all 5 `RaftReachable.step` hypotheses fully discharged — no abstract axioms remain" (REF-L17:S3, research/references/REF-L17-raft-lean-squad.md; also 02 C-8)
- Side B: CRITIQUE.md (same date): "`hqc_preserved` | **Conditionally discharged** | CPS13 via `CandidateLogCovers`" and "`HLogConsistency` ... is still taken as a conditional hypothesis"; code: "`ValidAEStep` contains `hqc_preserved` as a structure field (`ConcreteProtocolStep.lean`)" (REF-L17:S4, REF-L17:S1); the report "says all hypotheses are discharged while still listing a \"reduce trust boundary\" item" (08:C06, research/08-academic-literature-and-community.md)
- Why it matters for the debate: The raft-rs model safety theorem is conditional or unconditional depending on this point.
- Status: open
- Recorded in: 02 C-8; REF-L17 §13; 08 C06 entry

### CF-017 raft-lean-squad theorem and file counts
- Sub-claims: H2
- Type: number mismatch
- Side A: REPORT: "716 theorems, 79 Lean files" (01:S22, research/01-lean4-protocol-modeling.md; REF-L17:S3; 08:C06); CORRESPONDENCE.md: "681 theorems, 539+ `#guard`, 75 Lean files, 27 correspondence targets" (01:S22); blog: "443 theorems across 23 files" (REF-L17:S2)
- Side B: "repo scan: 80 files and 779 `theorem`/`lemma` tokens" (01:S22 [CODE]); CRITIQUE "769 theorem declarations" and "My static count is 769 `theorem`/`lemma` lines" (REF-L17:S4, REF-L17:S1, research/references/REF-L17-raft-lean-squad.md)
- Why it matters for the debate: Scale figures for an AI-generated verification effort differ by document and by counting method, including between two static counts in this corpus (779 vs 769).
- Status: open
- Recorded in: 01 C4; REF-L17 §13; 08 C06 entry

### CF-018 raft-lean-squad toolchain version
- Sub-claims: H2
- Type: number mismatch
- Side A: REPORT/CRITIQUE "Lean 4.30.0-rc2" (REF-L17:S3, REF-L17:S4, research/references/REF-L17-raft-lean-squad.md; 01:S22)
- Side B: "`lean-toolchain` file `leanprover/lean4:v4.28.0`" (REF-L17:S1; 01:S22); 08:C06 records "no `lean-toolchain` file found" (research/08-academic-literature-and-community.md)
- Why it matters for the debate: Which Lean version checked the proofs is needed to reproduce the result.
- Status: open
- Recorded in: 01 C4; REF-L17 §13; 08 C06 entry

### CF-019 raft-lean-squad #guard test count
- Sub-claims: H2, H3
- Type: number mismatch
- Side A: "REPORT 625" (REF-L17:S3, research/references/REF-L17-raft-lean-squad.md; 08:C06 "625 `#guard` tests"); CORRESPONDENCE "539+ `#guard`" (01:S22, research/01-lean4-protocol-modeling.md)
- Side B: "CRITIQUE 642 (+54 in proof files). My grep count is 696 total." (REF-L17:S4, REF-L17:S1)
- Why it matters for the debate: The size of the model-to-Rust correspondence test suite is reported inconsistently.
- Status: open
- Recorded in: 01 C4; REF-L17 §13

### CF-020 raft-lean-squad blog "zero sorry" versus interim critique
- Sub-claims: H2
- Type: doc-vs-doc
- Side A: blog claim "zero sorry" for `raftReachable_safe` (REF-L17:S2, research/references/REF-L17-raft-lean-squad.md)
- Side B: the same blog quotes an interim critique: "`raftReachable_safe` (RT2) is conditionally correct only: it proves safety *given* the hypotheses, but the hypotheses themselves are axioms rather than proved facts." (REF-L17:S2)
- Why it matters for the debate: "sorry-free" and "conditional on axiomatised hypotheses" describe the same theorem differently.
- Status: open
- Recorded in: REF-L17 §13

### CF-021 Veil case-study count
- Sub-claims: H2
- Type: number mismatch
- Side A: 16, "All 16 verified. Ivy failed on 2" (08:E01, research/08-academic-literature-and-community.md); "S1 reports 16 case studies" (REF-L04:S1, research/references/REF-L04-veil.md)
- Side B: "17 specifications of 15 different distributed protocols" (08:E03; REF-L04:S3); "the current repo lacks the CAV Rabia and SuzukiKasamiInts files and adds 29 model-checked TLA+ ports" (REF-L04:S4)
- Why it matters for the debate: Benchmark scale claims for Lean protocol verification depend on which suite version is cited.
- Status: open
- Recorded in: 08 C-3; REF-L04 §13 (08 C-3 also cites de Moura slides L14, a lead)

### CF-022 Veil performance relative to Ivy
- Sub-claims: H2
- Type: number mismatch
- Side A: "87.5% verified in under 15 seconds", all 16 verified (06:P10, FRO slide, research/06-lean4-language-limits-and-automation.md); "S1 reports Ivy timing out on 2 benchmarks that Veil verifies" (REF-L04:S1, research/references/REF-L04-veil.md)
- Side B: "Veil is more than 10x slower than Ivy ... 160 seconds, compared to Ivy's 50 seconds" (06:P02; REF-L04:S2)
- Why it matters for the debate: Performance at scale is central to the provability claim, and the two figures point in different directions.
- Status: open
- Recorded in: 06 C9; REF-L04 §13

### CF-023 Veil proof-reconstruction cost
- Sub-claims: H2
- Type: number mismatch
- Side A: "We keep it off by default since it's quite slow (>5x slowdown) and currently a bit brittle" (08:E13, Zulip, Apr 2025, research/08-academic-literature-and-community.md)
- Side B: "3-5x" (08:E01, 08:E02; 06:P02 "3-5x performance penalty", research/06-lean4-language-limits-and-automation.md)
- Why it matters for the debate: The cost of making SMT results kernel-checked determines whether foundational guarantees are practical by default.
- Status: open
- Recorded in: 08 C-2; 06 C9

### CF-024 Veil trust base: stated versus default
- Sub-claims: H2, D
- Type: claim-vs-code
- Side A: users "only have to trust the Veil frontend ..., the Lean compiler ..., and the Lean kernel" (REF-L04:S2, research/references/REF-L04-veil.md)
- Side B: "the shipped default `veil.smt.trust := true` (S4) also trusts the SMT solver unless the option is changed" (REF-L04:S4, same file)
- Why it matters for the debate: Whether a Veil proof is foundational depends on a default setting not reflected in the paper's trust statement.
- Status: open
- Recorded in: REF-L04 §13

### CF-025 Velvet automation: SMT-based or grind-based
- Sub-claims: H2
- Type: paper-vs-repo
- Side A: Velvet "combines automated SMT-based proofs" (REF-L06:S3 abstract, research/references/REF-L06-loom-velvet.md)
- Side B: "its evaluation used `grind` for all but one case and the current README says \"SMT-inspired proof automation\"" (REF-L06:S3, REF-L06:S4)
- Why it matters for the debate: The automation route affects both the trust base and reproducibility of Velvet results.
- Status: open
- Recorded in: REF-L06 §13

### CF-026 Velvet case-study count
- Sub-claims: H2
- Type: number mismatch
- Side A: "S1 reports 10 Velvet case studies" (REF-L06:S1, research/references/REF-L06-loom-velvet.md)
- Side B: "S3 reports 27 VERINA case studies; the repo's `CaseStudies/` has 61 files" (REF-L06:S3, REF-L06:S4)
- Why it matters for the debate: Evaluation scale differs by source.
- Status: open
- Recorded in: REF-L06 §13

### CF-027 Scaling of verification condition generation
- Sub-claims: H2
- Type: doc-vs-doc
- Side A: "Verification condition generation with proof assistants has not scaled so far… The same story holds in Lean." (06:P10, FRO NFM slide, research/06-lean4-language-limits-and-automation.md)
- Side B: "vcgen 0.32 s vs mvcgen 66 s on a 2,200-iteration loop-body benchmark, linear growth" (06:V04); mvcgen' ">100x on some synthetic benchmarks" (06:V05)
- Why it matters for the debate: Whether VC generation scales is a direct input to provability at realistic size; the counter-figures are synthetic and author-produced.
- Status: open
- Recorded in: 06 C2

### CF-028 Status and lineage of mvcgen
- Sub-claims: H2
- Type: doc-vs-doc
- Side A: "mvcgen is now deprecated" (06:V04, 4.35 notes, research/06-lean4-language-limits-and-automation.md)
- Side B: "`mvcgen'` is the experimental reimplementation" (06:V05, 06:V08); the NFM slide still lists "mvcgen — Lean's VC generator for monadic programs" (06:P10)
- Why it matters for the debate: Arguments citing "mvcgen" may refer to a deprecated or a current tool.
- Status: open
- Recorded in: 06 C7

### CF-029 LLM proof success rates for verified code
- Sub-claims: H2
- Type: number mismatch
- Side A: Verina: "best proof success 4.9% (o3) / 11.2% (Goedel-Prover-V2 32B), one trial" (06:A08, research/06-lean4-language-limits-and-automation.md)
- Side B: CLEVER audit: "original ceiling 1/161, audit stacks up to 96.2% end-to-end on sound-premise entries"; the audit's authors "argue the benchmark is flawed in 80/161 entries" (06:A09)
- Why it matters for the debate: AI-assisted proof capacity is cited for scale, and the figures differ by benchmark, model, date and success definition.
- Status: open
- Recorded in: 06 C8

### CF-030 CatCrypt handshake-security claims versus availability
- Sub-claims: H2
- Type: doc-vs-doc
- Side A: CatCrypt claims computational-model game-based proofs "covering a catalogue of constructions including TLS 1.3, SSH, WireGuard, Noise (IK/XX/KK/NK) and MLS" (02:C1, research/02-lean4-crypto-and-network-verification.md)
- Side B: the VCVio paper records the CatCrypt repository as "not publicly available" (02:V1); tls13-lean: "No security argument is claimed for confidentiality, authentication, or the handshake as a cryptographic protocol" (02:N1); qssh handshake security "Not verified" (02:N10)
- Why it matters for the debate: The only claimed Lean proofs of full handshake security could not be inspected, while the inspectable projects disclaim such proofs.
- Status: open
- Recorded in: 02 C-2

### CF-031 Hydrozoan: machine-checked versus "Read critically"
- Sub-claims: H2, D
- Type: interpretive
- Side A: safety and liveness are machine-checked "with no sorry and no axioms beyond the 3 standard ones"; "The proofs and helpers, a few thousand lines each, are AI-generated and never read by humans; the kernel checks them." (02:D4 paper, research/02-lean4-crypto-and-network-verification.md)
- Side B: README: "heavy LLM assistance" with "only human-plus-LLM review behind it. Read critically." (02:D4 [CODE])
- Why it matters for the debate: The weight of kernel-checked but unread AI proofs is framed differently within one project's own documents.
- Status: open
- Recorded in: 02 C-9

### CF-032 IronFleet proof-to-code ratio
- Sub-claims: H2
- Type: number mismatch
- Side A: implementation proof-to-code ratio "3.6 to 1" (04:S55, SOSP 2015, research/04-other-tools-protocol-verification.md)
- Side B: "Our ratio of proof annotation to implementation is 7.7:1 (5.4:1 if liveness proof annotations are excluded)" (04:S56, CACM 2017)
- Why it matters for the debate: Comparator proof-effort ratios are used to judge what realistic scale costs.
- Status: open
- Recorded in: 04 #1

### CF-033 IronFleet liveness and correctness scope
- Sub-claims: H2, H3, D
- Type: claim-vs-code
- Side A: "We prove complete functional correctness and its key liveness property" (REF-C02:C02-S1, research/references/REF-C02-ironfleet-verus.md); "the first methodology for automated machine-checked verification of the safety and liveness of non-trivial distributed system implementations" (04:S55, research/04-other-tools-protocol-verification.md); Grove: "IronFleet also includes a proof of liveness, though it does not handle thread concurrency" (REF-C10:S2, research/references/REF-C10-aneris-trillium-grove-fairneris.md)
- Side B: "A machine-checked proof that this obligation enables reduction is future work; instead, we sketch an informal argument" (REF-C02:C02-S1); "`assume TimeNotZeno(timefun);`" at `ViewAdvance.i.dfy:321` (REF-C02:C02-S5); Trillium describes IronFleet as one that "does not support node-local concurrency" and uses "a pen-and-paper argument for proving liveness" (04:S43); the same Trillium Related Work section is cited again in REF-C10 (REF-C10:S1, §8); Fairneris describes IronFleet as requiring each node to be "a simple single-threaded event loop", with liveness obtained by proving that event handlers terminate (REF-C10:S3)
- Why it matters for the debate: The main comparator's guarantee mixes machine-checked layers with an informal meta-argument and an assumed environment property.
- Status: open
- Recorded in: REF-C02 §13; REF-C10 §13 (Trillium, Fairneris and Grove items); 04 §9 (Trillium) and §10; REVIEW-phase1 "Unrecorded conflicts" (Trillium/IronFleet row)

### CF-034 IronKV line counts: Verus versus IronFleet
- Sub-claims: H2
- Type: number mismatch
- Side A: "Verus paper reports IronKV→Dafny proof at 8,070 lines and trusted 1,205" (REF-C02:C02-S2, research/references/REF-C02-ironfleet-verus.md)
- Side B: "IronFleet Figure 12 reports IronKV implementation-layer proof 2,937, impl 1,340, and spec 34 SLOC" (REF-C02:C02-S1)
- Why it matters for the debate: Cross-tool effort comparisons depend on consistent counting scopes.
- Status: open
- Recorded in: REF-C02 §13

### CF-035 Verdi Raft size
- Sub-claims: H2
- Type: number mismatch
- Side A: "530 lines of code and 50,000 lines of proof" (REF-C03:C03-S1, CPP 2016, research/references/REF-C03-verdi-raft.md)
- Side B: "Raft impl 520, proof 4,144 (linearizability layer only)" (REF-C03:C03-S2); "54K lines of Coq/OCaml" (REF-C03:C03-S4); "repo `.v` total is 50,190 lines" (REF-C03:C03-S3)
- Why it matters for the debate: The proof burden of a verified Raft is a common scale reference.
- Status: open
- Recorded in: REF-C03 §13

### CF-036 Tamarin 5G-AKA model and lemma counts
- Sub-claims: H2
- Type: paper-vs-repo
- Side A: model "roughly 500 LoC", lemmas "ca. 1000 LoC" with "124 different lemmas"; oracle "∼1000 LoC of Python" (REF-C05:C05-S3, research/references/REF-C05-tamarin.md)
- Side B: "`5G-AKA-nonBindingChannel/5G_AKA.spthy` has 2,771 lines and 80 `lemma` declarations, oracle 661 lines" (REF-C05 repository source [CODE], same file)
- Why it matters for the debate: Comparator effort figures are used to calibrate what is realistic for Lean.
- Status: open
- Recorded in: REF-C05 §13

### CF-037 TLS 1.3: Tamarin versus ProVerif findings and scope
- Sub-claims: H2
- Type: doc-vs-doc
- Side A: the ProVerif draft-18 analysis "did not uncover the potential mismatch between client and server view" (REF-C05:C05-S1, Cremers et al., research/references/REF-C05-tamarin.md)
- Side B: "In concurrent work by Bhargavan et al. [10], a similar issue was uncovered for the 0.5-round trip time (RTT) case." (REF-C05:C05-S1); Bhargavan et al. "did not consider" post-handshake authentication and state that the Tamarin work did not consider 0-RTT client-certificate authentication or 0.5-RTT data (REF-C06:C06-S1, research/references/REF-C06-proverif-cryptoverif.md)
- Why it matters for the debate: Mature tools' analyses are described with different coverage, which affects how comparator results are summarised.
- Status: open
- Recorded in: REF-C05 §13; REF-C06 §13

### CF-038 WireGuard identity mis-binding
- Sub-claims: H2
- Type: doc-vs-doc
- Side A: "the Donenfeld-Milner Tamarin analysis concluded there is no mis-binding even with all keys compromised" (as reported via REF-C06:C06-S5, research/references/REF-C06-proverif-cryptoverif.md; the Donenfeld-Milner paper is 07:F2, read in sections 1-4.3)
- Side B: Lipp et al. "disprove this property" under a precise Curve25519 model (REF-C06:C06-S5)
- Why it matters for the debate: The result of a symbolic proof depended on the precision of the cryptographic model.
- Status: open
- Recorded in: REF-C06 §13

### CF-039 WireGuard: "formally verified secure network tunnel protocol" versus key-exchange-only scope
- Sub-claims: H2, D
- Type: interpretive
- Side A: "The end result is a formally verified secure network tunnel protocol" (07:F2 abstract, research/07-protocol-spec-formats-and-corpora.md)
- Side B: the same paper formalises only the key exchange ("Leaving aside the cookie MAC system") (07:F2)
- Why it matters for the debate: Comparator headline claims can exceed the formalised scope.
- Status: open
- Recorded in: 07 C7

### CF-040 Noise Explorer pattern count
- Sub-claims: H2
- Type: paper-vs-repo
- Side A: "more than 57 handshake patterns" (REF-C06:C06-S6 abstract, research/references/REF-C06-proverif-cryptoverif.md)
- Side B: "The repo ships 59 pattern pairs (118 models)" (REF-C06:C06-S6r)
- Why it matters for the debate: Coverage figures for generated protocol models differ between paper and repository.
- Status: open
- Recorded in: REF-C06 §13

### CF-041 Tamarin soundness findings without maintainer response
- Sub-claims: H2, D
- Type: interpretive
- Side A: [S17] "reports false `verified` results on Tamarin 1.12.0 and other audit findings" (04:S17, research/04-other-tools-protocol-verification.md)
- Side B: "Tamarin operates in the symbolic model and thus can only capture attacks within that model ... Tamarin is not guaranteed to terminate." (04:S1); "Limitations: Risk of non-termination. Risk of false attacks." (04:S9); the documentation "does not respond to those findings" (04 #4)
- Why it matters for the debate: Trust in comparator tools bears on whether Lean's kernel-based trust is stronger or weaker.
- Status: open
- Recorded in: 04 #4

### CF-042 Chronos-Vault theorem counts
- Sub-claims: H2
- Type: number mismatch
- Side A: "184 theorems" and "\"78/78\" and \"778/78\"" (08:C17, research/08-academic-literature-and-community.md)
- Side B: "14/22 theorems proven; 42 theorems for 14 contracts" (08:C17, from the same repository's documents)
- Why it matters for the debate: Self-reported counts in a vendor repository are inconsistent with each other.
- Status: open
- Recorded in: 08 C-6

### CF-043 Clean: Poseidon BN254 primality sorry
- Sub-claims: H2
- Type: claim-vs-code
- Side A: "`instance : Fact (Nat.Prime BN254_PRIME)` is declared with `by sorry`" (`Clean/Circomlib/Poseidon.md:162`); the Poseidon blog (2026-05-04) calls primality "the only remaining `sorry`" (REF-L12, research/references/REF-L12-clean.md)
- Side B: "At `f8a4194`, `Clean/Specs/Poseidon.lean:27` instead proves it via CompPoly's Pratt certificate, and no such `sorry` remains." (REF-L12 [CODE])
- Why it matters for the debate: Documentation and code at a pinned commit disagree on whether the Poseidon proof is complete (the dossier itself calls the `.md` note stale).
- Status: open
- Recorded in: REF-L12 §13

### CF-044 SymCrypt kyber_terminates: proved or assumed
- Sub-claims: H2, H3
- Type: doc-vs-doc
- Side A: "It is therefore fully proved — not assumed." (`SampleNTTBridge.lean:108-110`, REF-L07:C1, research/references/REF-L07-aeneas-symcrypt.md); the paper says it is proved via Barbosa and Schwabe (REF-L07:A1 §7.2)
- Side B: almost-surely terminating with "no constructive bound is known, and the EasyCrypt ML-KEM proof leaves the same gap" (`VERIFIED.md:425-432`); "(out-of-scope, axiomatic) `kyber_terminates`" (`SampleNTTBridge.lean:304`) (REF-L07:C1)
- Why it matters for the debate: The termination status of ML-KEM sampling determines whether the end-to-end correctness claim carries an assumption.
- Status: open
- Recorded in: REF-L07 §13 item 1

### CF-045 SymCrypt NTT ring isomorphism and decryption-failure bounds
- Sub-claims: H2
- Type: paper-vs-repo
- Side A: "Our formal development proves that the specified NTT realizes the intended ring isomorphism and establishes the claimed decryption-failure bounds" (REF-L07:A1 §4, research/references/REF-L07-aeneas-symcrypt.md)
- Side B: "A grep of the public branch's `Spec/` (`RingEquiv`, `≃+*`, failure terms) found no such theorem" (REF-L07:C1); the paper says "We have released a significant subset" (REF-L07:A1)
- Why it matters for the debate: Claimed specification-level theorems could not be located in the released code.
- Status: open
- Recorded in: REF-L07 §13 item 3

### CF-046 VCVio: lattice schemes "future work" versus present in repo
- Sub-claims: H2
- Type: paper-vs-repo
- Side A: "ePrint 2026/899 §10 lists ML-KEM/ML-DSA/Falcon as \"future work\"" (REF-L10, research/references/REF-L10-vcvio.md)
- Side B: "the current repo contains `LatticeCrypto/` specifications and concrete implementations for all three. ... The repo still does not claim ML-KEM IND-CCA." (REF-L10 [CODE])
- Why it matters for the debate: Paper and repository describe different states of lattice-cryptography coverage in Lean.
- Status: open
- Recorded in: REF-L10 §13

### CF-047 Hopscotch pins an older VCVio
- Sub-claims: H2
- Type: number mismatch
- Side A: "Hopscotch README pins VCVio commit `e7f4c8d` and Lean v4.28.0" (REF-L10, research/references/REF-L10-vcvio.md)
- Side B: "VCVio HEAD is on v4.35.0" (REF-L10 [CODE])
- Why it matters for the debate: Results built on a library are not re-checked against its current version.
- Status: open
- Recorded in: REF-L10 §13

### CF-048 kairos-cedar: sorry-free versus one sorry
- Sub-claims: H2
- Type: claim-vs-code
- Side A: README line 5: a "sorry-free Lean soundness theorem" (05:D8, research/05-implementation-conformance-methods.md)
- Side B: `cedar-full/CedarFull/Soundness.lean`: "Total: 1 sorry, isolated to a single named blocker theorem", with "5 `sorry` occurrences" (05:D8)
- Why it matters for the debate: A completeness claim and the code disagree, possibly about different sub-project theorems.
- Status: open
- Recorded in: 05 #2

### CF-049 verified-consensus paper commit
- Sub-claims: H2
- Type: doc-vs-doc
- Side A: README "Paper: `consensus.tex` at commit `9f5ed717ffac`" (REF-L20, research/references/REF-L20-verified-consensus-iris-cslib.md)
- Side B: `docs/PROTOCOL.md` "Source: `consensus.tex` at commit `2f4b9d1c5e1a`" (REF-L20 [CODE])
- Why it matters for the debate: Which paper version the Lean development formalises is ambiguous.
- Status: open
- Recorded in: REF-L20 §13

### CF-050 Cedar forbid_trumps_permit statement
- Sub-claims: H2
- Type: paper-vs-repo
- Side A: "2024 sources show an explicit `∃ policy ∈ policies ∧ effect = forbid ∧ satisfied` premise" (REF-L01:S1, REF-L01:S4, research/references/REF-L01-cedar.md)
- Side B: "`theorem forbid_trumps_permit ... : (IsExplicitlyForbidden request entities policies) → (isAuthorized request entities policies).decision = .deny`" (REF-L01:C1, `Authorization.lean:69`; 02:A2, research/02-lean4-crypto-and-network-verification.md)
- Why it matters for the debate: A restated theorem may or may not have the same meaning (not checked).
- Status: open
- Recorded in: REF-L01 §13

### CF-051 Cedar typecheck theorem conclusion versus README wording
- Sub-claims: H2, D
- Type: definitional
- Side A: `typecheck_is_sound ... : ... typecheck policy env = .ok t → (∃ (b : Bool), EvaluatesTo policy.toExpr request entities b)` (REF-L01:C1, `Cedar/Thm/Typechecking.lean:47`, research/references/REF-L01-cedar.md)
- Side B: README: "either evaluation returns a value of that type, or it returns an error of type `entityDoesNotExist`, `extensionError`, or `arithBoundsError`" (02:A2, research/02-lean4-crypto-and-network-verification.md); the `EvaluatesTo` docstring "also allows for errors of type `entityDoesNotExist`, `extensionError`, and `arithBoundsError`" (REF-L01:C1, `Basic.lean:61`)
- Why it matters for the debate: A compact theorem conclusion hides a predicate that admits runtime errors; REVIEW-phase1 treats the scope as now explained and asks to "retain this as a model for similar hidden-predicate issues, not an outstanding contradiction".
- Status: open
- Recorded in: REVIEW-phase1 "Unrecorded conflicts" (Cedar row); REF-L01 §3

### CF-052 Cedar SymCC proof status
- Sub-claims: H2
- Type: paper-vs-repo
- Side A: "proofs of soundness and completeness using the Lean encoder are underway" (REF-L01:S2, OOPSLA 2024, research/references/REF-L01-cedar.md)
- Side B: "current repo contains them" (REF-L01:C1, REF-L01:S3)
- Why it matters for the debate: The state of the symbolic-compiler proofs depends on date.
- Status: open
- Recorded in: REF-L01 §13

### CF-053 SampCert proof size
- Sub-claims: H2
- Type: number mismatch
- Side A: "over 12,000 lines of proof" (REF-L02:S1, research/references/REF-L02-sampcert.md; 08:E11)
- Side B: "18,463 total Lean LOC at current commit" (REF-L02:C1)
- Why it matters for the debate: Effort figures differ by scope and date.
- Status: open
- Recorded in: REF-L02 §13

### CF-121 Telltale file, line and theorem totals
- Sub-claims: H2
- Type: number mismatch
- Side A: "CODE_MAP and inventory 767 files / 152,326 lines"; research/01 S15 "recorded 5,101 theorems, implemented_by 8, partial 1" (REF-L23:S8, research/references/REF-L23-telltale.md; 01:S15, research/01-lean4-protocol-modeling.md)
- Side B: "my lexical 830 / 164,896 (different scope)"; "my counts are 5,105, 14 attributes, 1 (different counting rules and/or commit)" (REF-L23 [CODE] @ c8c6a7f)
- Why it matters for the debate: Scale figures for the Telltale proof stack differ between its own inventory and two corpus counts.
- Status: open
- Recorded in: REF-L23 §13; 01 S15 entry

### CF-122 Perennial ArraySemantics soundness
- Sub-claims: H2, H3
- Type: interpretive
- Side A: "a design note dated 2026-10-24 records that it proved `False`" (REF-L22:S4, research/references/REF-L22-perennial-goose.md)
- Side B: "The 2026-10-25 decision removed the cause. At the inspected commit the defect is described as fixed, but no model instance was found to confirm consistency"; "`rg` found no `instance` of `go.Semantics`, `go.PreSemantics` or `ArraySemantics` in the repository" (REF-L22:S4)
- Why it matters for the debate: Go-program proofs in Perennial are conditional on typeclass assumptions that were once inconsistent and have no instance in the repository.
- Status: open
- Recorded in: REF-L22 §13 and §3a

### CF-128 Lean-SMT finite-field reconstruction count (paper-internal): 247 versus 348 of 403
- Sub-claims: H2
- Type: number mismatch
- Side A: introduction (page 2): "Out of 403 unsat QF_FF instances, CVC 5 with proof production solves 392, all verified by Carcara, while LEAN-SMT reconstructs 247." (06:A03, research/06-lean4-language-limits-and-automation.md; https://hanielbarbosa.com/papers/2026fmcad-ffproofs.pdf)
- Side B: §VI-B Results (page 10): "LEAN-SMT can solve and reconstruct 348 instances."; Table I row LEAN-SMT 348 solved / 24 TO / 31 MO of 403 total unsat; Table II covers "the 348 benchmarks solved by all configurations" (06:A03, same paper)
- Why it matters for the debate: Any lean-smt reconstruction rate for finite-field reasoning differs by about 25 percentage points depending on which figure in the same paper is cited.
- Status: open
- Recorded in: 06 C11 and A03; REVIEW-phase1 F-014 (per /root)

## H3 Implementation link

### CF-054 Leslie Rust verification files
- Sub-claims: H3
- Type: claim-vs-code
- Side A: "The README file tree lists `Leslie/Rust/CoreSemantics.lean` and `RuntimeSemantics.lean`" (01:S4, research/01-lean4-protocol-modeling.md; REF-L19:S1, research/references/REF-L19-leslie.md)
- Side B: "a tree listing at SHA 58e58b2502 shows no such files (only `docs/rust-verification*.md` and the `rust/` crate)" (01:S4); "The directory does not exist." (REF-L19)
- Why it matters for the debate: A documented Lean-to-Rust link may not exist in code.
- Status: open
- Recorded in: 01 C2; REF-L19 §13

### CF-055 tla-plus-lean-workflow: "Aeneas-verified" in abstract versus repo
- Sub-claims: H3
- Type: paper-vs-repo
- Side A: paper abstract: "$\mmap{3}$ is Aeneas-verified for ..." (REF-L18:S5, `paper-sttt/main.tex` line 114, research/references/REF-L18-leanda-tla-lean.md)
- Side B: "Repo: no Aeneas output, and the paper later says the LLM path is unproved." (REF-L18:S5)
- Why it matters for the debate: A claimed spec-to-Rust link is contradicted within the same artifact.
- Status: open
- Recorded in: REF-L18 §13

### CF-056 Can Rust code be verified against a Lean model
- Sub-claims: H3, H4
- Type: doc-vs-doc
- Side A: "tools for verifying Rust code are not (yet) up to the task" (02:A1, Cedar paper 2024, research/02-lean4-crypto-and-network-verification.md)
- Side B: "Our 237 KLOC Lean development establishes safety, panic-freedom, and functional correctness of 16.7 KLOC of Rust code supporting post-quantum cipher suites for x86-64 and ARM platforms." (02:M1, SymCrypt/Aeneas paper 2026)
- Why it matters for the debate: The two statements bracket whether a mainstream-language link exists; they differ in date and target.
- Status: open
- Recorded in: 02 C-1

### CF-057 LeanServer theorem and axiom counts
- Sub-claims: H2, H3
- Type: claim-vs-code
- Side A: "935 formal theorems", 0 sorry (02:N2 README, research/02-lean4-crypto-and-network-verification.md); THREAT_MODEL.md: "914 verified theorems (0 `sorry`, 0 `axiom`)" (05:D4, research/05-implementation-conformance-methods.md; REF-L14:S6); "Total: 164 provas, 0 axiomas" (`LeanServer/Proofs.lean:36`, REF-L14:S6, research/references/REF-L14-lean-http-stack.md)
- Side B: "929 theorem/lemma declarations, 0 sorry, 3 `admit`, 5 `axiom` lines, about 399 `native_decide`" at SHA 24b916aaa6 (02:N2 [CODE]); "the source has 5 `axiom` declarations" (05:D4); "929 theorem/lemma declarations and 5 `axiom` declarations in the source" (REF-L14:S6)
- Why it matters for the debate: A protocol server's axiom-free claim conflicts with its source.
- Status: open
- Recorded in: 02 C-4; 05 #1; REF-L14 §13

### CF-058 LeanServer "no spec-impl gap" versus FFI and threat model
- Sub-claims: H3
- Type: claim-vs-code
- Side A: README "sem spec-impl gap" ("no spec-impl gap") (REF-L14:S6, research/references/REF-L14-lean-http-stack.md; 02:N2)
- Side B: "`FFI.lean`'s optional OpenSSL backend, which is asserted (not proved) to be \"semantically equivalent\"", and `THREAT_MODEL.md` "lists the Lean compiler, C compiler and C FFI as unverified" (REF-L14:S6); "77 `extern`/`opaque` bindings with about 1,000 lines of C FFI" (02:N2)
- Why it matters for the debate: A Lean-as-implementation route still depends on unverified FFI code.
- Status: open
- Recorded in: 02 C-4; REF-L14 §13

### CF-059 quiche Lean theorem counts
- Sub-claims: H2, H3
- Type: number mismatch
- Side A: blog, Apr 2026: "518 theorems across 24 Lean files with only 3 sorry" (02:N3, research/02-lean4-crypto-and-network-verification.md; 07 C2); CRITIQUE.md (2026-05-20): "70 Lean files, 1348 theorems, 0 sorry" (07:L2, research/07-protocol-spec-formats-and-corpora.md); other documents "1,348 / 1,443 / 1,567" and Route-B totals "1,759+ / 2,822+ / 2,864+" (REF-L16:S5, research/references/REF-L16-ghostbuster-veridns-quiche-flare.md)
- Side B: "1,592 theorem/lemma declarations in 81 files, 0 sorry, 258 `native_decide`" at SHA 5a8195ebf9 (02:N3 [CODE]; 07:L2; REF-L16:S5 "1,592 by grep")
- Why it matters for the debate: The scale of the largest Lean QUIC model is reported inconsistently across snapshots.
- Status: open
- Recorded in: 02 C-5; 07 C2; REF-L16 §13

### CF-060 quiche toolchain version
- Sub-claims: H3
- Type: number mismatch
- Side A: "`lean-toolchain` is `v4.29.0`" (REF-L16:S5, research/references/REF-L16-ghostbuster-veridns-quiche-flare.md)
- Side B: "REPORT says \"Lean 4.29.1\"" (REF-L16:S5)
- Why it matters for the debate: Reproducibility depends on the toolchain actually used.
- Status: open
- Recorded in: REF-L16 §13

### CF-061 VeriDNS "verified to the RFC" versus assumed oracle
- Sub-claims: H3, D
- Type: interpretive
- Side A: "Verified to what spec? To the RFC itself!" (02:N7 README, research/02-lean4-crypto-and-network-verification.md)
- Side B: in the same README "the \"honesty oracle\" is assumed and the FFI transport and live network are outside the proof" (02:N7)
- Why it matters for the debate: A headline RFC-conformance claim rests on stated environment assumptions.
- Status: open
- Recorded in: 02 C-6

### CF-062 VeriDNS TCP scope
- Sub-claims: H3
- Type: claim-vs-code
- Side A: README: "TCP transport and TCP fallback. UDP only." (out of scope) and "Concurrent serving (single-threaded request loop)" (07:L4, research/07-protocol-spec-formats-and-corpora.md; REF-L16:S3, research/references/REF-L16-ghostbuster-veridns-quiche-flare.md)
- Side B: `resolveWithIO_sound` in `VeriDNS/Proof/NameTree.lean:2018` has an "`hnetTcp : NetworkConsistentTcp ...` hypothesis" (07:L4); "`Main.lean` has `tcpConnLoop`/`tcpServeLoop` and `Std.Mutex DnsCache`, `UdpSocket.lean` has `veri_dns_tcp_listen`/`_accept`" (REF-L16:S3)
- Why it matters for the debate: The scope of the verified DNS server differs between documentation and code.
- Status: open
- Recorded in: 07 C4; REF-L16 §13

### CF-063 VeriDNS ioResumeLoop_sound versus internal audit
- Sub-claims: H3
- Type: doc-vs-doc
- Side A: `ioResumeLoop_sound` gives that "every answer the running server returns is one the model authorizes" (REF-L16:S3 README, research/references/REF-L16-ghostbuster-veridns-quiche-flare.md)
- Side B: `docs/audit/w4-summary.md` "lists the unwitnessed `WorldModels` pack (sev 5) and `primeRootHints` breaking capstones \"at the real entry point\" (sev 5)" (REF-L16:S4)
- Why it matters for the debate: The project's own audit questions whether the end-to-end theorem applies to the running entry point.
- Status: open
- Recorded in: REF-L16 §13

### CF-064 LeanMqtt payload status
- Sub-claims: H3
- Type: paper-vs-repo
- Side A: payloads are "left as future work" (REF-L13:S1, SBLP 2026 paper, research/references/REF-L13-leanmqtt.md; 07:P-MQTT); "the code's `Payload` is an empty stub (`Payload/Basic.lean:6-11`)" (REF-L13:S2)
- Side B: README: "the parser is sound and complete" (REF-L13:S2); "repo contains `Payload.roundtrip` at `LeanMqtt/Packets/Payload/Proofs.lean:9`" (07:L5, research/07-protocol-spec-formats-and-corpora.md)
- Why it matters for the debate: Two corpus readings of the same repository disagree on whether payloads are modelled, which changes the scope of the parser guarantee.
- Status: open
- Recorded in: 07 C5; REF-L13 §13

### CF-065 LeanMqtt conformance statement count
- Sub-claims: H3
- Type: number mismatch
- Side A: "251 mandatory conformance statements" (REF-L13:S1 §6, research/references/REF-L13-leanmqtt.md)
- Side B: "`conformance.md` contains 285 occurrences of `MQTT-` identifiers" (REF-L13:S3)
- Why it matters for the debate: Conformance coverage claims depend on how statements are counted.
- Status: open
- Recorded in: REF-L13 §13

### CF-066 Lean Std.Http parser changes since snapshot
- Sub-claims: H3
- Type: doc-vs-doc
- Side A: FINDINGS: master `src/Std/Http` "has had no parser changes since 2026-07-24" (REF-L14:S2, research/references/REF-L14-lean-http-stack.md)
- Side B: "Lean master at 2026-11-06 contains `optionalUnlessEof`" (REF-L14:S1); the PR page showed #15841 as Open, "which may be stale cache" (REF-L14:S5)
- Why it matters for the debate: Verification of a moving upstream parser may describe a superseded version.
- Status: open
- Recorded in: REF-L14 §13

### CF-067 Native-evaluation compiler trust versus SampCert's extractor route
- Sub-claims: H3, D
- Type: interpretive
- Side A: native-evaluation axioms record proofs depending on "the correctness of the entire compiler, and not just on the much smaller `kernel`." (03:E01, Lean reference "Axioms", research/03-lean4-code-extraction-and-fidelity.md)
- Side B: SampCert "Custom code extractor: the deployed artifact does not go through the Lean compiler. TCB includes the extractor, not the compiler." (03:E12, NFM 2026 slides)
- Why it matters for the debate: Which tools enter the trusted base differs by extraction route.
- Status: open
- Recorded in: 03 #1

### CF-068 Aeneas and Verity "zero axioms/sorry" versus raw greps
- Sub-claims: H3
- Type: claim-vs-code
- Side A: "The compiler proof stack carries 0 axioms and no `sorry`." (03:E26, Verity README, research/03-lean4-code-extraction-and-fidelity.md); Aeneas likewise states "zero axioms/sorry" for its proof stack (03 #3)
- Side B: "Crude counts over all `.lean`: sorry 29, admit 19, axiom 12" (03:E26 [CODE]); "crude greps over the same repos find matches" (03 #3)
- Why it matters for the debate: Scoped claims and whole-repository counts can be confused when weighing trust bases.
- Status: open
- Recorded in: 03 #3

### CF-069 hax Lean backend maturity
- Sub-claims: H3, H4
- Type: doc-vs-doc
- Side A: "The new Lean backend does not use the hax engine. Instead, the `cargo-hax` command line tool drives Charon and Aeneas"; backends listed include `legacy-lean` (03:E17, research/03-lean4-code-extraction-and-fidelity.md); hax README labels Lean (via Aeneas) "Recommended for Lean" (REF-L08:C1, REF-L08:B1, research/references/REF-L08-hax-libcrux.md)
- Side B: the Lean backend of hax is "under active development", citing "independent analysis" of semantic gaps (03:E18)
- Why it matters for the debate: Maturity of a second Rust-to-Lean route affects generality claims.
- Status: open
- Recorded in: 03 #7; REF-L08 §13 item 1 (the sorryfree.com side of that item is an excerpt and is listed in the inadmissible section)

### CF-070 hax-lean and hax version lines
- Sub-claims: H3
- Type: number mismatch
- Side A: "provencrypt pins `hax-lean v0.3.30` and Lean `v4.31.0`" (REF-L08:C2, research/references/REF-L08-hax-libcrux.md)
- Side B: "its Cargo pins hax 0.4.2 per B2"; "hax `main` proof library is on `v4.35.0`" (REF-L08:B2, REF-L08:C1)
- Why it matters for the debate: Reproducing a Rust-to-Lean result requires knowing which pipeline versions were combined.
- Status: open
- Recorded in: REF-L08 §13 item 2 (the B5 "targets Lean v4.31.0" side is an excerpt and is listed in the inadmissible section)

### CF-071 hax and libcrux repository homes
- Sub-claims: H3
- Type: doc-vs-doc
- Side A: "hax docs link `github.com/pereq-dev/hax` and the Lean library requires `pereq-dev/aeneas`"; "libcrux docs and A1 cite `celabshq/libcrux`" (REF-L08:C1, REF-L08:C3, research/references/REF-L08-hax-libcrux.md)
- Side B: "provencrypt uses `cryspen/hax-lean` and `cryspen/aeneas`"; "`cryspen/libcrux` also resolves" (REF-L08:C2, REF-L08:C3)
- Why it matters for the debate: Provenance of the pinned toolchain is ambiguous.
- Status: open
- Recorded in: REF-L08 §13 item 3

### CF-072 hax `implies` semantics
- Sub-claims: H3
- Type: doc-vs-doc
- Side A: "B2 (2026-10-12) says `implies` is not lazy" (REF-L08:B2, research/references/REF-L08-hax-libcrux.md)
- Side B: hax CHANGELOG Unreleased "adds a lazy `implies!` and says the Rust semantics of `implies` \"is no longer reversed\"" (REF-L08:C1)
- Why it matters for the debate: A semantic bug in the translation layer bears on the fidelity of Rust-to-Lean specifications.
- Status: open
- Recorded in: REF-L08 §13 item 4

### CF-073 Aeneas support for unsafe and union code
- Sub-claims: H3, H4
- Type: doc-vs-doc
- Side A: Charon extracted the unsafe/union code "with zero changes"; "Aeneas does not support union types, and the deserializer is built on exactly that" (08:E18, research/08-academic-literature-and-community.md)
- Side B: "Aeneas has limited support for unsafe Rust and does not support raw pointers, interior mutability, or concurrency" (08:E07); "Aeneas does not have much support for unsafe code; but unchecked accesses such as the one above are modeled like regular accesses" (08:E19)
- Why it matters for the debate: The supported Rust subset bounds which codebases the main Lean route reaches.
- Status: open
- Recorded in: 08 C-9

### CF-074 Aeneas supported loop and borrow subset
- Sub-claims: H3, H4
- Type: doc-vs-doc
- Side A: Aeneas README (main): "`return` inside nested loops, or `break`/`continue` to outer loops are not supported yet" (03:E14, research/03-lean4-code-extraction-and-fidelity.md; REF-L07:C2, research/references/REF-L07-aeneas-symcrypt.md)
- Side B: `docs/user/src/README.md`: "loops: no nested loops for now", "no functions pointers/closures", "no nested borrows in function signatures" (REF-L07:C2); the paper extended Aeneas with "a large class of nested (mutable) borrows" (REF-L07:A1 §1)
- Why it matters for the debate: Documents in one repository give different support boundaries.
- Status: open
- Recorded in: REF-L07 §13 item 4

### CF-075 Aeneas Result semantics across revisions (paper, pinned d71d2e3f, main)
- Sub-claims: H3
- Type: paper-vs-repo
- Side A: the paper describes the Lean `Result` as an exception monad with `←` propagating failure (REF-L07:A1 §5.1, research/references/REF-L07-aeneas-symcrypt.md); at the revision SymCrypt pins, `backends/lean/Aeneas/Std/Primitives.lean:63-67` defines "`inductive Result (α : Type u) where | ok (v: α): Result α | fail (e: Error): Result α | div`", with "`spec := theta`" (REF-L07:C4, Aeneas @ d71d2e3f, [CODE])
- Side B: "Aeneas `main` (c5eb4a2, Lean v4.35.0) has since replaced this with an interaction-tree `Result` (`abbrev Result := ITree RustEffect`) and a `DWP`-based `spec` at `WP.lean:56`" (REF-L07:C2); the dossier adds: "That main-branch definition is not the one SymCrypt's proofs were checked against and is recorded here only to show that the two revisions differ" (REF-L07 §3.1)
- Why it matters for the debate: What a SymCrypt `⦃ ⦄` theorem asserts depends on which Aeneas revision's `Result` and `spec` are cited.
- Status: open (reframed). The earlier gap, that the pinned revision had not been inspected, is closed by evidence: REF-L07 now inspects d71d2e3f (C4). The entry is kept as a version and provenance difference between the pinned and main revisions, as REVIEW-phase1 asks ("preserve as a version/provenance limitation, not as demonstrated semantic disagreement"). The paper's exception-monad description and the pinned `ok | fail | div` inductive are not in tension in the dossier's account (REF-L07 §3.1: "it matches the paper's stated intent").
- Recorded in: REF-L07 §13 item 6 and §3.1; REVIEW-phase1 "Unrecorded conflicts" (Aeneas row)

### CF-076 SymCrypt paper Appendix C versus code
- Sub-claims: H3
- Type: paper-vs-repo
- Side A: "Paper Appendix C shows the `.MemoryAllocationFailure =>` branch empty" (REF-L07:A1, research/references/REF-L07-aeneas-symcrypt.md)
- Side B: "code has `out_of_memory` (Decaps.lean:1726-1730)" (REF-L07:C1)
- Why it matters for the debate: Minor; published listings may differ from the checked code (the dossier suggests an HTML rendering loss).
- Status: open
- Recorded in: REF-L07 §13 item 5

### CF-077 SymCrypt ML-KEM size
- Sub-claims: H3
- Type: number mismatch
- Side A: "Paper Fig. 4: Rust 2,349, properties plus proofs 36,352" (REF-L07:A1, research/references/REF-L07-aeneas-symcrypt.md)
- Side B: "README: Rust 2,125, properties and proofs 37,989" (REF-L07:C1)
- Why it matters for the debate: Proof-to-code ratios for the main Rust-to-Lean result differ by source.
- Status: open
- Recorded in: REF-L07 §13 item 2

### CF-078 Cedar Rust line counts
- Sub-claims: H3
- Type: number mismatch
- Side A: "Rust production code 24,915 LOC (Evaluator/Authorizer 13,664, Validator 11,251)" (REF-L01:S4, AWS blog, research/references/REF-L01-cedar.md)
- Side B: "Rust prod 15,693 with separate Rust tests 20,458 and parser 4,114" (REF-L01:S1, FSE paper)
- Why it matters for the debate: The size of the implementation linked by differential testing depends on counting rules.
- Status: open
- Recorded in: REF-L01 §13

### CF-079 Cedar Lean versus Rust execution speed
- Sub-claims: H3
- Type: number mismatch
- Side A: "On average, Lean takes 5 microseconds ... compared to 7 microseconds for Rust" (REF-L01:S4, research/references/REF-L01-cedar.md)
- Side B: "median execution time for the Lean authorizer is 6 microseconds, compared to 10 microseconds for Rust" (REF-L01:S1)
- Why it matters for the debate: Performance of executable Lean models is cited for using Lean as a test oracle.
- Status: open
- Recorded in: REF-L01 §13

### CF-080 Cedar bug counts
- Sub-claims: H3, H5b
- Type: number mismatch
- Side A: "4 proof-found + 21 DRT/PBT" (REF-L01:S1, research/references/REF-L01-cedar.md)
- Side B: "nearly two dozen bugs" (REF-L01:S2; 08:E10, research/08-academic-literature-and-community.md)
- Why it matters for the debate: Bug yield from proofs versus testing is evidence for H5b.
- Status: open
- Recorded in: REF-L01 §13

### CF-081 SampCert trusted C++ size
- Sub-claims: H3
- Type: number mismatch
- Side A: "adding only 57 lines of C++ to our trusted computing base" (08:E11, research/08-academic-literature-and-community.md; REF-L02:S1, research/references/REF-L02-sampcert.md)
- Side B: "`ffi.cpp` 69 lines at commit `a2c53bb`" (REF-L02:C1)
- Why it matters for the debate: The size of the trusted extraction boundary is a key trust figure.
- Status: open
- Recorded in: REF-L02 §13

### CF-082 LNSym proved listing versus ELF test listing
- Sub-claims: H3
- Type: claim-vs-code
- Side A: the proved `program` uses linked addresses, e.g. "`0xd0000463` adrp" (REF-L03:C1, research/references/REF-L03-lnsym.md)
- Side B: the ELF test listing from `.o` has "`0x90000003`"; "it means the ELF test is not a mechanical check of the proved listing" (REF-L03:C1)
- Why it matters for the debate: The link between proved machine code and the shipped object is not mechanically checked.
- Status: open
- Recorded in: REF-L03 §13 item 2

### CF-083 pg-lean "no opaque" versus grep
- Sub-claims: H3
- Type: claim-vs-code
- Side A: the client inventory contains "no `opaque`" (REF-L11:S2 README, research/references/REF-L11-tls13-lean.md)
- Side B: "static grep finds 9 raw `opaque` tokens and 3 `implemented_by` attributes in `Pg/` (the latter replace kernel-checked definitions at runtime)" (REF-L11 [CODE])
- Why it matters for the debate: `implemented_by` can make running code differ from verified definitions.
- Status: open
- Recorded in: REF-L11 §13

### CF-084 tls13-lean RFC 9846 versus RFC 8446
- Sub-claims: H3
- Type: doc-vs-doc
- Side A: README headline "implementing RFC 9846" (REF-L11, research/references/REF-L11-tls13-lean.md)
- Side B: "the key-schedule spec and identifiers are RFC 8446 §7.1; README explains RFC 9846 obsoletes 8446 with unchanged labels" (REF-L11 [CODE])
- Why it matters for the debate: Which normative text the Lean spec follows matters for spec fidelity.
- Status: open
- Recorded in: REF-L11 §13

### CF-085 Clean: "behave exactly as you described" versus trust boundaries
- Sub-claims: H3, D
- Type: doc-vs-doc
- Side A: site claim "guaranteed to behave exactly as you described" (REF-L12:S4, research/references/REF-L12-clean.md)
- Side B: blog trust boundaries: "hand-modelled Circom, unverified export, unproved `poseidon1Opt = poseidon1`" (REF-L12:S3)
- Why it matters for the debate: Marketing language and stated trust boundaries describe different guarantees.
- Status: open
- Recorded in: REF-L12 §13

### CF-086 miTLS verification against RFC 8446
- Sub-claims: H3
- Type: doc-vs-doc
- Side A: "miTLS has been verified against the formal TLS 1.3 specification (RFC 8446) and against computational security models" (REF-C01:C01-S4, sota.io, research/references/REF-C01-everest.md); "a verified reference implementation of the TLS protocol" (REF-C01:C01-S3, mitls.org)
- Side B: "we did not complete verifying our *implementation* of the TLS handshake" and "The proof of the TLS-1.3 handshake in miTLS-fstar was left incomplete" (REF-C01:C01-S1; 04:S53, research/04-other-tools-protocol-verification.md); "a complete verification of the TLS 1.3 handshake ... as future work" (REF-C01:C01-S2); `--admit_smt_queries true` in `MiTLS.Handshake.fst` (REF-C01:C01-S5)
- Why it matters for the debate: The flagship comparator for verified TLS has a contested completion status.
- Status: open
- Recorded in: REF-C01 §13; 04 #3 (04's aggregator side was a lead; the admissible Side A comes from REF-C01)

### CF-087 miTLS deployment in Firefox NSS
- Sub-claims: H3
- Type: doc-vs-doc
- Side A: miTLS code "runs in Firefox's NSS library" (REF-C01:C01-S4, research/references/REF-C01-everest.md)
- Side B: C01-S1 "names Firefox only among deployments of Everest crypto (HACL*/EverCrypt) and says miTLS handshake was deployed in msquic, later superseded by Schannel" (REF-C01:C01-S1)
- Why it matters for the debate: Deployment claims are used as evidence that verified code reaches production.
- Status: open
- Recorded in: REF-C01 §13

### CF-088 HACL* size figures within the Everest retrospective
- Sub-claims: H3
- Type: number mismatch
- Side A: "212,000 F* LOC / 101,000 C LOC (HACL* section)" (REF-C01:C01-S1, research/references/REF-C01-everest.md)
- Side B: "245,000 F*+Vale LOC / 135,000 C+asm LOC (overview)" (REF-C01:C01-S1)
- Why it matters for the debate: Scale figures within one source differ, probably by inclusion of Vale.
- Status: open
- Recorded in: REF-C01 §13

### CF-089 Verdi "verified implementation" versus shim bugs
- Sub-claims: H3, D
- Type: doc-vs-doc
- Side A: "verified implementation ... runs on real networks" (REF-C03:C03-S1, research/references/REF-C03-verdi-raft.md)
- Side B: "Bugs V1-V8 causing crashes, data loss and command injection in the deployed system" (REF-C03:C03-S4); "the bugs lie in the trusted shim, which the authors list as TCB" (REF-C03:C03-S2); Fonseca et al.: "No protocol bugs were found in the verified systems" (04 §7, research/04-other-tools-protocol-verification.md)
- Why it matters for the debate: A verified implementation's guarantee stops at its trusted shim.
- Status: open
- Recorded in: REF-C03 §13

### CF-090 s2n-tls TLS version scope
- Sub-claims: H3
- Type: paper-vs-repo
- Side A: the handshake proof covers "a subset of TLS 1.2" (REF-C04:C04-S2, CAV 2018, research/references/REF-C04-fiat-crypto-aws-saw.md)
- Side B: a TLS 1.3 Cryptol spec "according to RFC 8446" and a SAW check `tls13rfcSimulatesS2N `{16}`, "bounded to 16-message traces" (REF-C04:C04-S3)
- Why it matters for the debate: The comparator's scope depends on date and on a bounded check.
- Status: open
- Recorded in: REF-C04 §13

### CF-091 aws-lc verification scope
- Sub-claims: H3
- Type: doc-vs-doc
- Side A: AWS aims for "verification of full functional correctness" for commonly used algorithms (REF-C04:C04-S6, research/references/REF-C04-fiat-crypto-aws-saw.md)
- Side B: the README "lists a subset with caveats and has HKDF, ECDSA, ECDH, and EC key rows commented out of the \"Verified Code\" table" (REF-C04:C04-S4)
- Why it matters for the debate: Aims and verified coverage differ for a production comparator.
- Status: open
- Recorded in: REF-C04 §13

### CF-092 RefTLS: extracted model versus public repo
- Sub-claims: H3
- Type: paper-vs-repo
- Side A: "the paper reports an 800-line ProVerif model extracted from the code" (REF-C06:C06-S1, research/references/REF-C06-proverif-cryptoverif.md)
- Side B: "The public repo contains only hand-written ProVerif models and no extracted model or compiler" (REF-C06:C06-S2r)
- Why it matters for the debate: A claimed code-to-model link could not be found in the public artifact.
- Status: open
- Recorded in: REF-C06 §13

### CF-093 Noise Explorer generated-code size
- Sub-claims: H3
- Type: paper-vs-repo
- Side A: "IKpsk2 Go at 341 lines" (REF-C06:C06-S6, Fig. 8, research/references/REF-C06-proverif-cryptoverif.md)
- Side B: "The current repo file has 537 lines" (REF-C06:C06-S6r)
- Why it matters for the debate: Generated implementations may have changed since the paper's analysis.
- Status: open
- Recorded in: REF-C06 §13

### CF-094 Ivy QUIC targeted draft version
- Sub-claims: H3
- Type: doc-vs-doc
- Side A: "The currently targeted version is IETF draft 18" (REF-C07:S3, `microsoft/ivy` README, research/references/REF-C07-ivy-quic.md)
- Side B: "Ivy comes with a QUIC specification (based on IETF draft 23)" (REF-C07:S4, Galois), referring to a `quic23` branch of `kenmcmil/ivy` (REF-C07:S5)
- Why it matters for the debate: Which spec version the tester checks against changes what conformance means.
- Status: open
- Recorded in: REF-C07 §13

### CF-095 Ivy QUIC number of implementations tested
- Sub-claims: H3
- Type: number mismatch
- Side A: "four implementations tested, one abandoned early" (REF-C07:S1, SIGCOMM, research/references/REF-C07-ivy-quic.md)
- Side B: "We applied the testing methodology to HTTP servers based on the four chosen implementations" (REF-C07:S2, SECDEV)
- Why it matters for the debate: The dossier records the count as consistent, but only three were tested substantively per S1.
- Status: open
- Recorded in: REF-C07 §13

### CF-096 Ivy QUIC error classification denominators
- Sub-claims: H3, H5b
- Type: number mismatch
- Side A: "13 spec-violation detections out of 27" (REF-C07:S1, Table 1, research/references/REF-C07-ivy-quic.md)
- Side B: "adverse stimulus (78%) or ... compliance violation (57%) or both" over 23 root-caused errors (REF-C07:S2)
- Why it matters for the debate: Mixing denominators would misstate the testing yield.
- Status: open
- Recorded in: REF-C07 §13

### CF-097 QUIC draft-29 paper attribution (correction with provenance)
- Sub-claims: H3
- Type: doc-vs-doc
- Side A: research/05 labelled arXiv 2503.01374 as a "Galois ... draft-29 update" (as recorded in REF-C07 §13, research/references/REF-C07-ivy-quic.md; the label was not found in research/05-implementation-conformance-methods.md when this register was written)
- Side B: "The PDF read in full is by Crochet, Rousseaux, Sambon, Piraux, Legay (EPIQ'21), not Galois" (REF-C07:S6)
- Why it matters for the debate: This is a correction with provenance rather than a substantive disagreement; REVIEW-phase1 asks that the attribution be corrected and its provenance retained.
- Status: open
- Recorded in: REF-C07 §13; REVIEW-phase1 "Unrecorded conflicts" (QUIC row)

### CF-098 MongoDB trace validation: new bug or known violation
- Sub-claims: H3, H5b
- Type: doc-vs-doc
- Side A: Davis et al. applied trace validation "discovering a non-trivial implementation bug" (REF-C08:S1 §9, REF-C08:S3 §5, research/references/REF-C08-tla-trace-validation.md)
- Side B: the violation was "a known violation of the specification" and "had we not already known about this behavior, it would have been a significant discovery" (REF-C08:S2)
- Why it matters for the debate: The bug yield of trace validation is evidence for checking code without proofs.
- Status: open
- Recorded in: REF-C08 §13

### CF-099 etcd trace validation bug yield
- Sub-claims: H3, H5b
- Type: doc-vs-doc
- Side A: Kuppe's slides list etcd-raft item "#149" (inefficiency) (REF-C08:S7, research/references/REF-C08-tla-trace-validation.md)
- Side B: PR #113 author: "No. I did not find bug so far." (REF-C08:S6)
- Why it matters for the debate: As above, for a second system.
- Status: open
- Recorded in: REF-C08 §13

### CF-100 CCF: TLA+ trace validation in CI versus Lean replay at HEAD
- Sub-claims: H3, H5b
- Type: paper-vs-repo
- Side A: "the NSDI paper (2025) describes TLA+ trace validation as part of CI" (REF-C08:S1, research/references/REF-C08-tla-trace-validation.md)
- Side B: "at commit 6404230 (2026-11-06) the TLA+ tree is gone and Lean replay is the CI check" (REF-C08:S5)
- Why it matters for the debate: The production status of the trace-validation comparator changed to a Lean route.
- Status: open
- Recorded in: REF-C08 §13

### CF-101 CCF Lean: model inclusion aim versus replay limit
- Sub-claims: H3, H5b
- Type: doc-vs-doc
- Side A: AGENT.md: "B(implementation) \subseteq B(Model)" (REF-L21:S1, research/references/REF-L21-ccf-lean.md)
- Side B: PR #8584: a pass is "not universal C++ refinement" (REF-L21:S5)
- Why it matters for the debate: Trace replay against a Lean model is described both as an inclusion goal and as a non-refinement check.
- Status: open
- Recorded in: REF-L21 §13

### CF-102 CCF PR #8663 removals versus files at HEAD
- Sub-claims: H3
- Type: claim-vs-code
- Side A: PR #8663 "Removes TLA-only trace tooling: the `consistency_trace_validation` e2e test, `tests/tvc.py`, the `samples/apps/basic_tv` app" (REF-L21:S3, research/references/REF-L21-ccf-lean.md)
- Side B: "At HEAD `tests/consistency_trace_validation.py`, `tests/tvc.py`, `samples/apps/basic_tv/` and the CMake test `consistency_trace_validation` exist and feed the Lean `consistency_traces` job" (REF-L21:S1)
- Why it matters for the debate: Which tooling backs the Lean consistency check is unclear.
- Status: open
- Recorded in: REF-L21 §13

### CF-103 CCF Lean package name
- Sub-claims: H3
- Type: doc-vs-doc
- Side A: "PR #8663 refers to `lean/ccf-raft`" (REF-L21:S3, research/references/REF-L21-ccf-lean.md)
- Side B: "at HEAD the package is `lean/consensus` (renamed in #8657)" (REF-L21:S1, REF-L21:S4)
- Why it matters for the debate: Minor; affects locating the cited code.
- Status: open
- Recorded in: REF-L21 §13

### CF-104 CCF replay assurance-limits pointer
- Sub-claims: H3
- Type: doc-vs-doc
- Side A: `lean/consensus/README.md` says replay's "assurance limits are documented there" (in `replay/README.md`) (REF-L21:S1, research/references/REF-L21-ccf-lean.md)
- Side B: "that file at HEAD has no section on assurance limits; the detailed limits are in the PR #8584 description" (REF-L21:S1, REF-L21:S5)
- Why it matters for the debate: The stated limits of the Lean replay are not where the repository says they are.
- Status: open
- Recorded in: REF-L21 §13

### CF-105 ShardStore validation overhead: 12% versus 13%
- Sub-claims: H3, H5b
- Type: number mismatch
- Side A: "the combined property definitions and test harnesses comprise 12% of the ShardStore code base" (REF-C09:S1 §1, research/references/REF-C09-aws-lightweight-fm.md)
- Side B: "These artifacts combined are only 13% of the total code base and 20% of the size of the implementation code" (REF-C09:S1 §8.2)
- Why it matters for the debate: The cost of lightweight conformance checking is a comparison point for Lean routes.
- Status: open
- Recorded in: REF-C09 §13

### CF-106 PObserve stream engine
- Sub-claims: H3
- Type: doc-vs-doc
- Side A: "Stream Processing functionality built using Apache Flink" (REF-C09:S7, HPTS 2022 slides, research/references/REF-C09-aws-lightweight-fm.md)
- Side B: "The stream processing block in the figure is implemented using Java Streams" (REF-C09:S3)
- Why it matters for the debate: Minor; describes a changed implementation of a runtime-monitoring comparator.
- Status: open
- Recorded in: REF-C09 §13

### CF-107 PObserve instrumentation requirement
- Sub-claims: H3, H5b
- Type: doc-vs-doc
- Side A: PObserve "does not require any additional instrumentation" (REF-C09:S3, research/references/REF-C09-aws-lightweight-fm.md)
- Side B: it requires "a developer-written parser and structured logs containing the needed events" (REF-C09:S3, `logparser.md`)
- Why it matters for the debate: One statement understates the effort needed to check code against a spec at runtime.
- Status: open
- Recorded in: REF-C09 §13

### CF-108 Learned-model testing: no unexpected QUIC behaviour versus bugs elsewhere
- Sub-claims: H3, H5b
- Type: interpretive
- Side A: A4 "reports no unexpected QUIC behaviour" (05:A4, research/05-implementation-conformance-methods.md)
- Side B: "A1, A3, A5 report bugs in other protocols" (05:A1, 05:A3, 05:A5)
- Why it matters for the debate: The yield of model-learning methods varies by target; the recording file notes the targets differ.
- Status: open
- Recorded in: 05 #6

### CF-123 Perennial default branch
- Sub-claims: H3
- Type: doc-vs-doc
- Side A: `AGENTS.md`: "GitHub's default branch is `master`" (REF-L22:S6, research/references/REF-L22-perennial-goose.md)
- Side B: "`git ls-remote --symref` on the access date reports `HEAD` → `refs/heads/lean`" (REF-L22 [CODE])
- Why it matters for the debate: Minor; which branch is cited determines whether Lean or Rocq Perennial is being discussed.
- Status: open
- Recorded in: REF-L22 §13

### CF-124 Wording of the CompCert guarantee
- Sub-claims: H3, D
- Type: doc-vs-doc
- Side A: "the executable code it produces is proved to behave exactly as specified by the semantics of the source C program" (03:E38, research/03-lean4-code-extraction-and-fidelity.md; quoting the CompCert manual)
- Side B: the same manual says the theorem takes C to "abstract syntax trees for the assembly-level Asm language (before assembling and linking)", and that assembly and linking have "no formal guarantees yet" (REF-C11:C11-S1 §1.2, §1.3, research/references/REF-C11-verified-compilation.md)
- Why it matters for the debate: The reference point for verified compilation carries an unverified assembly and linking boundary that the headline wording omits.
- Status: open
- Recorded in: REF-C11 §13; 03 E38

### CF-125 CakeML in-logic bootstrapping versus CompCert extraction trust
- Sub-claims: H3, D
- Type: interpretive
- Side A: CakeML §11 contrasts CakeML's in-logic bootstrapping with CompCert's reliance on extraction and OCaml (REF-C11:C11-S4, research/references/REF-C11-verified-compilation.md)
- Side B: "no bugs due to the use of axioms for interfacing untrusted code, or the use of the extractor to OCaml, were found." (REF-C11:C11-S3 §7)
- Why it matters for the debate: How much an extraction step weakens a compilation guarantee is argued differently by the two sources.
- Status: open
- Recorded in: REF-C11 §13

### CF-127 Aeneas and Charon soundness: trusted translators versus pen-and-paper metatheory with a repaired flaw
- Sub-claims: H3
- Type: doc-vs-doc
- Side A: "We trust that Charon correctly extracts MIR into LLBC and that Aeneas correctly translates LLBC into Lean" (REF-L07:A1 §3, research/references/REF-L07-aeneas-symcrypt.md)
- Side B: "The proofs of the article by Ho et al. are only done with pen and paper; sometimes, the authors only sketch the reasoning over large case analyses" (REF-L07:A6 §1); the mechanisation "found a flaw in the proofs of some of their fundamental lemmas. We repair their methodology using techniques from rewriting theory" (REF-L07:A6); a "lacking argument ... could have invalidated their strategy" (REF-L07:A6 §1, as quoted in REF-L07 §13 item 7)
- Why it matters for the debate: The trust placed in the Rust-to-Lean translation rests on a metatheory that the paper lists as a trust assumption and that a secondary source describes as partly informal and since repaired.
- Status: open
- Recorded in: REF-L07 §13 item 7 and §6

## H4 Generality

### CF-109 Aeneas port onto SymM: in progress or future
- Sub-claims: H4
- Type: doc-vs-doc
- Side A: "Aeneas, the tool Signal Shot will lean on for `libsignal` verification conditions, is being ported onto it now." (08:E17, Apr 2026, research/08-academic-literature-and-community.md)
- Side B: SymM "is used to implement the verification-condition generator, Velvet, and, in the future, Aeneas" (08:E21, roadmap, Sep 2026)
- Why it matters for the debate: The timeline of a shared verification backend affects claims about a general Rust-to-Lean route.
- Status: open
- Recorded in: 08 C-11

## H5 Inference (H5a, H5b)

### CF-110 Conformance direction in Lean projects
- Sub-claims: H5b, D
- Type: definitional
- Side A: "Etheorem is tested against external vectors (Lean as implementation under test)" (05:D1, research/05-implementation-conformance-methods.md)
- Side B: "chapulin and Cedar use Lean as the oracle for another implementation" (05:D2; 02:A1, research/02-lean4-crypto-and-network-verification.md)
- Why it matters for the debate: Lean appears on opposite sides of a conformance check, so "checked against a Lean spec" can mean two different things.
- Status: open
- Recorded in: 05 #5

## D Definitions

### CF-111 "Verified": kernel-checked theorem versus differential-testing link
- Sub-claims: D, H3, H5b
- Type: definitional
- Side A: lean-zip, Perennial and Aeneas use "verified" for theorem-level kernel-checked statements (03:E14, 03:E34, 03:E37, research/03-lean4-code-extraction-and-fidelity.md); SymCrypt uses Aeneas translation of Rust: "we then use Aeneas to translate LLBC into an equivalent Lean model" (02:M1, research/02-lean4-crypto-and-network-verification.md)
- Side B: julia-in-lean: the model "mirrors the Julia source by hand and is checked against the real implementation by differential testing, not by a verified compiler or a verified semantics of Julia" (03:E33); Cedar: "we use extensive differential random testing to confirm that our Rust code and Lean model agree" (08:E10, research/08-academic-literature-and-community.md; 02:A1); EVMYulLean validated by conformance tests "22,330 of 22,332" (02:Z3); flare links Mojo to Lean "by hashes plus differential tests" (02:N4); raft-lean-squad "is a hand-written model of raft-rs checked against the Rust by tests (correspondence), not an extraction" (08:C06)
- Why it matters for the debate: The sources do not share a name for proof-level versus test-level links, which is the core of D.
- Status: open
- Recorded in: 02 C-10; 03 #5; 08 C-10

### CF-112 Meaning of "refinement of an implementation" and "foundational"
- Sub-claims: D, H3
- Type: definitional
- Side A: Trillium is "the first foundationally verified proof that a concrete implementation of a distributed protocol correctly implements an abstract TLA+ specification" (04:S43, research/04-other-tools-protocol-verification.md)
- Side B: IronFleet links spec to code through "three layers: high-level spec, distributed-protocol layer, implementation layer, with refinement between them" in Dafny (04:S55); Igloo/SCION link specifications to code "with different trust bases (SMT-based or mixed provers)" (04:S50, 04:S51; S50 is recorded as abstract only)
- Why it matters for the debate: "Refinement" and "foundational" are used with different trust bases across comparators.
- Status: open
- Recorded in: 04 #5

### CF-113 Specification-based testing as verification (Ivy)
- Sub-claims: D, H5b
- Type: definitional
- Side A: "our specification is not an abstract implementation of the protocol, but is instead an executable monitor"; "27 errors in the server implementations"; presented as testing, not proof (04:S30, research/04-other-tools-protocol-verification.md)
- Side B: "Compared to proof assistants, Ivy provides domain-specific support for decidable proof automation... On the other hand, Ivy relies on a vastly larger trusted computing base than typical proof assistants." (04:S31)
- Why it matters for the debate: Whether spec-driven testing counts as verifying an implementation depends on the definition adopted for D.
- Status: open
- Recorded in: 04 #6

### CF-114 Specification fidelity: what Lean does not check
- Sub-claims: D, H3
- Type: interpretive
- Side A: "What Lean can't check is whether the theorem says what you think it says." (08:E12, abstract record, research/08-academic-literature-and-community.md); "researchers must still manually check that the specification faithfully captures the intended protocol and its assumptions" (08:E06)
- Side B: model-versus-code agreement is measured by testing: "we use extensive differential random testing to confirm that our Rust code and Lean model agree" (08:E10); SymCrypt leaves "three questions for review: is it the right formal specification, is it the right set of properties for this code, and what trust assumptions do they rely on?" (02:M1; 08:E07); Signal Shot: "We need your eyes on the definitions before anyone proves anything." (08:E17)
- Why it matters for the debate: The recording file notes no source claims Lean checks fidelity; the sources differ on how the gap is closed (human review versus testing).
- Status: open
- Recorded in: 08 C-12

## Cross-cutting

### CF-115 Size of the Lean trusted base
- Sub-claims: H2, H3, D
- Type: interpretive
- Side A: the TCB is "the Lean kernel and the Lean compiler", "much smaller than trusting a full SMT solver" (06:A01, bv_decide paper, research/06-lean4-language-limits-and-automation.md); the kernel is described as small and the elaborator as untrusted (06:E06–E08)
- Side B: postmortems report soundness bugs in the kernel and runtime and say `lake build` "does not protect users from adversarial proofs" (06:P12, 06:P13); GMP 6.1.2 "contained a bug that could be exploited to construct a proof of `False`" (08:E15, research/08-academic-literature-and-community.md); these "matter when checking proofs that arrive from an untrusted source, not for a development that only elaborates its own code" (06:V08)
- Why it matters for the debate: Every Lean-based claim inherits this trust base, and its practical size depends on `native_decide`/`bv_decide` use.
- Status: open
- Recorded in: 06 C1

### CF-116 trustCompiler history
- Sub-claims: H3, D
- Type: doc-vs-doc
- Side A: 2023 Zulip: "Scott Morrison cites lean4#2654 adding an axiom that `Lean.reduceBool` and `Lean.reduceNat` depend on" (03:E10, research/03-lean4-code-extraction-and-fidelity.md)
- Side B: "the `Lean.trustCompiler` machinery was removed from Lean in version 4.35.0" (03:E02; the corpus marks this wording as paraphrased); "The `native_decide` tactic creates a bespoke axiom for each invocation" (03:E01)
- Why it matters for the debate: How native-evaluation trust is recorded has changed across versions, which affects how `#print axioms` output is read.
- Status: open
- Recorded in: 03 #2

### CF-117 Evidence policy: meaning of [FULL]
- Sub-claims: D (affects admissibility for all sub-claims)
- Type: definitional
- Side A: "The central rule requires the full source to be fetched and read" (REVIEW-phase1 F-002, research/REVIEW-phase1.md; rule in research/CONVENTIONS.md)
- Side B: "research/06 explicitly defines FULL as whole-document retrieval plus reading relevant sections, including cases restricted to abstracts/evaluation lines, keyword-matched lines, or a truncated document" (REVIEW-phase1 F-002, on research/06-lean4-language-limits-and-automation.md); 08:E12 is tagged "[FULL for the abstract record only ... the body is NOT read]" (research/08-academic-literature-and-community.md)
- Why it matters for the debate: Admissibility of every entry in this register depends on the depth label; CONVENTIONS now adds [SECT] as a claim-scoped level and topic files are being re-tagged (review F-002).
- Status: open
- Recorded in: REVIEW-phase1 "Unrecorded conflicts" (FULL row); REVIEW-phase1 F-002

### CF-118 Trace-validation and lightweight-FM claims resting partly on excerpts
- Sub-claims: H3, H5b
- Type: interpretive
- Side A: "Trace-validation and lightweight-FM claims based partly on inadmissible search excerpts" (REVIEW-phase1 "Unrecorded conflicts", research/REVIEW-phase1.md)
- Side B: required handling: "Treat as provisional leads until F-001 is fixed; a conflict ledger is not a route around evidence admission." (REVIEW-phase1). The REF-C08 and REF-C09 entries in this register cite only sources tabled as [FULL] or [CODE] in those dossiers' current source tables.
- Why it matters for the debate: H5b evidence from trace validation and runtime monitoring must not rest on excerpts.
- Status: open
- Recorded in: REVIEW-phase1 "Unrecorded conflicts" (trace-validation row); REVIEW-phase1 F-001

### CF-119 Catalogue yes/partial cells versus dossier qualifications
- Sub-claims: H2, H3, D
- Type: interpretive
- Side A: "Catalogue property yes/partial cells versus dossier-specific proof/test/model qualifications" (REVIEW-phase1 "Unrecorded conflicts", research/REVIEW-phase1.md)
- Side B: required handling: "Address F-013; the summaries must not silently strengthen or conceal the underlying results." (REVIEW-phase1; F-013 "The generated catalogue collapses assurance levels and mixed links")
- Why it matters for the debate: Arguments citing catalogue cells may overstate what the underlying dossiers establish.
- Status: open
- Recorded in: REVIEW-phase1 "Unrecorded conflicts" (catalogue row); REVIEW-phase1 F-013

## Conflicts resting on unverified leads (inadmissible)

These conflicts were recorded in the corpus, but at least one side rests on a search excerpt, a snippet, an unread or partially read source, or an earlier-phase statement that is not itself a source. They carry no CF number and Phase 2 should ignore them as evidence.

| Recorded in | Conflict | Inadmissible side |
|---|---|---|
| 01 C5; REF-L20 §13 | CSLib Raft port claimed versus no Raft content in CSLib at `76b7943` | Claim rests on a Phase 1 secondary source and on a BaDaaS README snippet (REF-L20:S4) |
| 02 C-7 (part) | Veil liveness per SoK arXiv 2608.21935 | SoK fetched truncated, partially read (02 leads) |
| 08 C-1 (part) | Veil liveness "future work" | Older search-result snippet, L20, not re-fetched |
| 03 #4 | Loom POPL 2026 / Velvet CAV 2026 venue and status | ACM page behind an access wall (lead L12) |
| 03 #6 | lean-zip trusted FFI zlib versus pure-Lean README | Phase 1 secondary summary |
| 04 #2 | Verifpal bounded sessions versus unbounded-session claim | 2020 Verifpal paper seen only as a snippet |
| 04 #3 (part) | miTLS "verified against RFC 8446" per aggregator pages | Aggregator pages are leads; the admissible version of this conflict is the miTLS entry under H3 |
| 05 #3 | Ellsberg bug count 10 | Earlier snippet |
| 05 #4 | ProFuzzBench 10 protocols versus 13 servers | Repository README not fully read |
| 07 C1; REF-L15 §13 | Earlier "no Lean 4 models for TLS/QUIC/DNS" gap line and earlier lists of libp2p-fv/formal-eth as Lean network work versus repositories | Side A is an earlier-phase corpus statement, not a source |
| 07 C3 | Eywa 33 bugs versus "32 unique bugs" | 33 comes from a Phase 1 search excerpt |
| 07 C6 | Signal ProVerif attribution: blog versus 2016 paper | Blog not re-fetched |
| 08 C-4 | Signal Shot announced 2026-04-20 versus 2026-04-21 | Conference slide L14 is a lead |
| 08 C-5 | raft-rs bugs: "no implementation bugs" versus 3 bugs | Both sides are leads (L19, L16) |
| 08 C-7 | Veil stars 366 versus 350 | 350 from a cached page not in the source register |
| 08 C-8 | Kernel bug-hunt totals: "5 kernel + 2 runtime" versus E15 PR list | Side A is a Phase 1 summary, not a source |
| REF-L03 §13 item 1 | `bv_decide` handles LNSym ARM verification versus SHA-512 theorems ending in `sorry` | Release-notes side, REF-L03:S2, is a snippet |
| REF-L08 §13 item 1 (part) | hax Lean backend "not yet complete" | sorryfree.com search excerpt |
| REF-L08 §13 item 2 (part) | hax 0.4 "targets Lean v4.31.0" | Cryspen blog B5 is a search excerpt |
| REF-L08 §13 item 3 (part) | Organisational rename "A new name for the next chapter" | Search excerpt only |

## Recorded items not carried as conflicts

Listed for completeness, without a verdict. These items appear in a Conflicts section but the recording file itself states the two sources agree, or no second side is given:
- REF-L02 §13 item 3 (SampCert deployment path; "Consistent, recorded for completeness").
- REF-L05 §13 ("None found between sources"; "small library" versus 12,004 lines is noted as a characterisation difference).
- REF-C08 §13 item 1 (CCF 15 log statements; "Consistent").
- REF-L16 §13 last item (the name "VeriDNS" also names a non-Lean verifier; a name collision, the other work not read).
- REF-L21 §13 item 5 (C++ trace sink can drop events; whether a drop can make a replay pass is an open question with no opposing source).
