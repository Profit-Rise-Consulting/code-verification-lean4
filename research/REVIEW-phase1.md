# Independent skeptical review of Phase 1

## Summary

This review assesses whether the corpus can support a strong council debate, not whether the owner's hypothesis is true. The corpus contains substantial, useful information: theorem statements, implementation boundaries, source-level caveats, negative results, independent replications, and limitations that many surveys would omit. It is materially better than an excerpt collection. Nevertheless, the first-pass snapshot is not yet ready for an unqualified evidence freeze because some inadmissible excerpts still support claims, source-depth labels are inconsistent, and several framing and semantic ambiguities could decide the debate before the advocates begin.

Findings identified across the review: **1 blocker, 10 major, 2 minor**. These are historical finding counts, not a claim that all remain unresolved at the final recheck. These are findings about evidence and debate design, not votes for either side. The orchestrator should record dispositions rather than silently delete inconvenient findings. The earlier model-family allocation problem was rechecked and is fixed in the current council kit.

Final review snapshot: 2026-11-07. Coverage includes all eight topic files and all 29 dossiers: REF-L01–L08, L10–L21, and REF-C01–C09; L09 is intentionally unassigned. The generated reference catalogue was also reviewed because it is a principal council entry point. New dossiers were checked after they landed rather than counted as omissions.

Rechecks during finalisation: research/06's specification-free inference and percentage error were corrected; research/07 removed EXT entries and now explains PART scopes; C08/C09 report full rereads of their previously excerpt-only sources; Cedar now expands `EvaluatesTo` and explains additional semantic caveats. The reviewer independently fetched and read both Cryspen sources behind L08 B4/B5 in full, confirming the substantive statements and supplying corrected publication dates to the orchestrator. At the last local inspection, L08's source table still called these search excerpts; that remaining admission/metadata repair is narrow and does not require new research. The evidence-depth policy and final summary-table issues still require explicit disposition.

## Method

- **Independence:** The reviewer is from the OpenAI model family, distinct from the Claude authors of the corpus. No ruling on H1–H5 or the compound motion is made.
- **Requirements:** Read the original requests, conventions, dossier template, and council kit first. Compared source-admission practices with the owner's explicit rejection of excerpt-only evidence and request for analysed, generalised prior-work references.
- **Full-text review:** Used full local text, including structured full-text extraction assistance, to review the topic files and available dossiers. Manually reread decisive passages, theorem predicates, source tables, conflict sections, and the council rules. Extraction assistance identifies audit leads; it does not substitute for a primary-source check in the ledger.
- **Independent checks:** Fetched the public sources listed below independently of the authors' local notes. Checked both positive claims and limitations. Pinned raw files were preferred for code-sensitive questions; floating README checks establish only what that page states, not a historical commit's complete proof closure.
- **Limits:** No Lean build, execution, compilation, or axiom-dependency audit was performed. No git command was run. A source-access failure is marked unverifiable, not wrong; a README confirmation is not independent proof that its code builds or has the claimed semantics.
- **Outcome language:** “Confirmed” means the precise, bounded claim is supported by the fetched source. “Wrong” means a fetched source contradicts it. “Unverifiable” means this check did not establish it. These labels do not grade the entire project.

## Spot-check ledger

The ledger deliberately includes some qualifications and claim interpretations that should not be promoted into stronger conclusions. URLs identify what was actually fetched; local locations are finding aids, not external citations.

| Check | Claim checked | Location | Source | Result |
|---|---|---|---|---|
| S01 | Cedar uses proofs about its Lean model plus differential random testing against Rust, not a Rust refinement proof. | REF-L01 §§2,5 | [Cedar paper](https://arxiv.org/html/2407.01688v1) | Confirmed. The authors explicitly describe proof-guided model development and empirical model/implementation comparison. |
| S02 | SampCert's sampler/DP proofs do not establish constant-time security or eliminate its deployment FFI boundary. | REF-L02 §§2,5,8 | [SampCert paper](https://arxiv.org/html/2412.01671v1) | Confirmed. C++ `getRandomByte` FFI and the constant-time limitation are explicit. |
| S03 | Veil defaults to trusting SMT results; disabling trust requests proof reconstruction. | REF-L04 §6 | [Pinned Veil Base.lean](https://github.com/verse-lab/veil/blob/57447722e7f25f06c75cff8735d10d41697afc14/Veil/Base.lean) | Confirmed. `veil.smt.trust` has `defValue := true`. This is not evidence that every benchmark was reconstructed. |
| S04 | Lentil offers temporal/fairness reasoning but no demonstrated source-language implementation frontend. | REF-L05 §§3,5 | [Lentil repository](https://github.com/verse-lab/Lentil) | Confirmed as bounded README inspection. Absence from that description is not a universal absence proof. |
| S05 | tls13-lean explicitly disclaims handshake/state-machine security proofs and trusts HACL C FFI. | REF-L11 §§2,5 | [tls13-lean repository](https://github.com/pb64-lean/tls13-lean) | Confirmed. Its stated proofs concern parsing, serialisation and byte-level laws, not end-to-end TLS security. |
| S06 | LeanMQTT proves roundtrips while packet payloads remain work in progress. | REF-L13 §§2,4 | [LeanMQTT repository](https://github.com/edusporto/lean-mqtt) | Confirmed as author-stated scope. |
| S07 | LeanServer reports 935 theorems and zero sorry; these counts do not alone establish protocol security. | REF-L14 §§2,6 | [LeanServer repository](https://github.com/AfonsoBitoque/LeanServer) | Confirmed as README self-report only, not an independently reproduced count or build. |
| S08 | stdhttp-verify targets a patched Std.Http copy and states AI assistance. | REF-L14 §2 | [stdhttp-verify repository](https://github.com/eumemic/stdhttp-verify) | Unverifiable in this independent fetch. Retain the authors' separate code-inspection evidence, but do not describe this row as confirmation. |
| S09 | libp2p-fv describes a proposal and proposed obligations, not a completed verified implementation. | REF-L15 §§2,3 | [libp2p-fv repository](https://github.com/NyxFoundation/libp2p-fv) | Confirmed. The distinction is explicit. |
| S10 | raft-lean-squad's headline 716-theorem/end-to-end claim is not itself a compiler-extracted Rust refinement. | REF-L17 §§2,5 | [raft-lean-squad repository](https://github.com/dsyme/raft-lean-squad) | Confirmed as bounded repository description. The headline requires the premise analysis in S29. |
| S11 | The Ben-Or account reports four to five days, human intervention, and AI cheating. | REF-L18 §§2,8 | [Protocols Made Fun post](https://protocols-made-fun.com/proofs-are-closer.html) | Unverifiable: fetch returned no page content. |
| S12 | Leslie describes a shallow TLA embedding and distributed examples; this does not establish its Rust correspondence. | REF-L19 §§2,5 | [Leslie repository](https://github.com/rupakm/leslie) | Confirmed as repository scope; no Rust refinement proof established by this check. |
| S13 | The Everest retrospective says TLS handshake implementation verification was not completed. | REF-C01 §2 | [Everest retrospective](https://dl.acm.org/doi/full/10.1145/3805702) | Confirmed. This must not be conflated with its record-layer results. |
| S14 | Verdi Raft proves safety/linearizability with 90 invariants, but not liveness. | REF-C03 §§2,8 | [Verdi Raft paper](https://homes.cs.washington.edu/~mernst/pubs/raft-proof-cpp2016.pdf) | Confirmed. The authors explicitly leave liveness to future work. |
| S15 | IronFleet proves safety and liveness through Dafny refinement, with explicit environmental assumptions. | REF-C02 §§2,8 | [IronFleet paper](https://www.microsoft.com/en-us/research/wp-content/uploads/2015/10/ironfleet.pdf) | Confirmed. Fair scheduling, failure and communication assumptions matter. |
| S16 | The Verus IronKV port verifies the host program but does not reproduce the protocol-layer proof. | REF-C02 §§3,8 | [verified-ironkv repository](https://github.com/verus-lang/verified-ironkv) | Confirmed as an author-stated limitation. |
| S17 | Tamarin-to-code work translates models into I/O separation-logic specifications and verifies Go/Java implementations conditionally on model assumptions. | REF-C05 §2 | [Arquint et al. abstract page](https://arxiv.org/abs/2212.04171) | Unverifiable for the detailed claim: this fetch exposed the abstract, not the relevant proof and code sections. A title or abstract is insufficient. |
| S18 | The Signal/ProScript paper does not formally establish translator soundness. | REF-C06 §§2,5 | [Signal/ProScript paper](https://bblanche.gitlabpages.inria.fr/publications/KobeissiBhargavanBlanchetEuroSP17.pdf) | Confirmed. The authors explicitly say resulting ProVerif proofs do not necessarily imply proofs for source code. |
| S19 | Noise Explorer does not establish functional correctness/security of generated implementation code. | REF-C06 §§5,8 | [Noise Explorer paper](https://eprint.iacr.org/2018/766.pdf) | Unverifiable in this fetch; no usable page content returned. |
| S20 | `native_decide` can introduce bespoke axioms and extend trust to native computation/compiler correctness. | research/03 E01 | [Lean reference: Axioms](https://lean-lang.org/doc/reference/latest/Axioms/) | Confirmed. Zero textual `axiom` declarations is not zero theorem dependencies. |
| S21 | Lean's C ABI FFI is not a foreign-code proof; even its data-transfer surface has restrictions. | research/03 E03 | [Lean FFI reference](https://lean-lang.org/doc/reference/latest/Run-Time-Code/Foreign-Function-Interface/) | Confirmed. The page notes C structs cannot be passed/returned by value; interoperability alone supplies no foreign-code theorem. |
| S22 | Kani's soundness is conditional on unwinding checks and documented limitations. | research/04 S63 | [Kani soundness documentation](https://model-checking.github.io/kani/soundness.html) | Unverifiable: source-access failure. |
| S23 | Kani does not verify concurrent code. | research/04 S65 | [Kani comparison documentation](https://model-checking.github.io/kani/tool-comparison.html) | Confirmed as the documentation's stated capability boundary. |
| S24 | TLS state fuzzing found previously unknown issues in GnuTLS, JSSE and OpenSSL. | research/05 A1 | [TLS state-fuzzing paper](https://www.cs.ru.nl/~erikpoll/papers/tls_fuzzing.pdf) | Confirmed. Bug discovery is empirical evidence, not universal formal conformance. |
| S25 | An independent study found eight bugs in Verdi's unverified shim/runtime and none in its verified protocol core. | REF-C03 §9 | [Distributed-system bug study](https://www.cs.purdue.edu/homes/pfonseca/papers/eurosys2017-dsbugs.pdf) | Confirmed within the study's scope. “None found” is not a proof that the core contains no bugs. |
| S26 | RFC 9846 is the current TLS 1.3 document used by the corpus. | research/07 L1; REF-L11 §1 | [RFC 9846 information page](https://www.rfc-editor.org/info/rfc9846) | Confirmed. A surprising identifier was checked rather than rejected from memory. |
| S27 | SymCrypt's Lean verification relies on extraction/translation trust and does not establish leakage resistance. | REF-L07 §§2,5 | [SymCrypt paper](https://arxiv.org/html/2609.15648v1) | Confirmed. The public release and experimental algorithm sets must remain distinct. |
| S28 | hax has a Lean backend, but backend availability does not equal full libcrux protocol verification. | REF-L08 §§2,5 | [hax repository](https://github.com/hacspec/hax) | Confirmed as framework scope. |
| S29 | raft-lean-squad's valid concrete step assumes quorum preservation. | REF-L17 §3 | [Pinned ConcreteProtocolStep.lean](https://raw.githubusercontent.com/dsyme/raft-lean-squad/ab7a184fbdfe22ac3b894e92791fb9f324e2ccbc/formal-verification/lean/FVSquad/ConcreteProtocolStep.lean) | Confirmed. `ValidAEStep` contains `hqc_preserved`; a downstream theorem does not independently discharge that field. |
| S30 | The raft artifact pins v4.28.0 despite an adjacent report mentioning v4.30.0-rc2. | REF-L17 §§1,13 | [Pinned lean-toolchain](https://raw.githubusercontent.com/dsyme/raft-lean-squad/ab7a184fbdfe22ac3b894e92791fb9f324e2ccbc/formal-verification/lean/lean-toolchain) | Confirmed. The retrieved file is exactly `leanprover/lean4:v4.28.0`. |
| S31 | Leslie's simulation soundness is not a completed proof. | REF-L19 §6 | [Pinned Refinement.lean](https://raw.githubusercontent.com/rupakm/leslie/58e58b2502b2a570e4ce4fdb6b40748c3cb30e5d/Leslie/Refinement.lean) | Confirmed. `SimulationRel.soundness` has a literal `sorry` body. |
| S32 | Cedar's typecheck theorem guarantees a Boolean with no runtime-error alternative. | REF-L01 §3, if read without predicate expansion | [Pinned Typechecking.lean](https://raw.githubusercontent.com/cedar-policy/cedar-spec/a4c5a89cd15637afc1dab39ef0c89baf31c7ebdf/cedar-lean/Cedar/Thm/Typechecking.lean) | Wrong under that interpretation. `EvaluatesTo` permits `entityDoesNotExist`, `extensionError`, and `arithBoundsError`; the theorem excerpt is verbatim but its predicate needs explanation. research/02 already states the exception correctly. |
| S33 | Lentil's specific pinned Basic.lean contains the stated fairness operators. | REF-L05 §§3,12 | [Attempted pinned Basic.lean](https://raw.githubusercontent.com/verse-lab/Lentil/1ddf1fc/Lentil/Basic.lean) | Unverifiable: that path/revision was not accessible. This does not contradict S04's weaker README-supported claim. |
| S34 | Trace validation checks compatibility, not universal trace refinement, and insufficient logging can accept erroneous traces. | REF-C08 §§2,5 | [Trace-validation paper](https://arxiv.org/html/2404.16075v2) | Confirmed. Nonempty compatibility of specification/trace behaviors is weaker than inclusion of all implementation traces. |
| S35 | CCF finite replay establishes compatibility with an enabled model execution, not universal C++ refinement. | REF-L21 §§5,8 | [CCF PR 8584](https://github.com/microsoft/CCF/pull/8584) | Confirmed. The source explicitly states the finite-trace and recorded-field boundary. This is substantive production checking evidence without an end-to-end C++ proof. |
| S36 | libcrux's guarantees cover verified modules only, and Lean was described as experimental proof work. | REF-L08 B4 | [Cryspen: strengths and limits](https://cryspen.com/post/strengths-and-limitations/) | Confirmed by full-page read. Published February 12, 2026, not the dossier's October 3 date. Also explicitly disclaims compiled-executable side-channel proofs. |
| S37 | hax 0.4 makes Charon+Aeneas the main Lean backend and targets Lean 4.31.0 with known soundness bugs. | REF-L08 B5 | [Cryspen: hax 0.4](https://cryspen.com/post/hax_0_4_release/) | Confirmed by full-page read. Published October 1, 2026, not the dossier's October 3 date. It describes tested Rust core models, not a fully verified translator. |
| S38 | Clean soundness conditions the user specification on user assumptions and modeled constraints, with completeness separate. | REF-L12 §3 | [Pinned Formal.lean](https://raw.githubusercontent.com/Verified-zkEVM/clean/f8a4194c604488cb732024890aebd083968222ab/Clean/Circuit/Formal.lean) | Confirmed. `FormalCircuit` bundles separate soundness/completeness obligations; this source does not establish backend-export correspondence. |
| S39 | The current s2n TLS 1.3 simulation check is invoked at length 16 and excludes waits in trace matching. | REF-C04 §3 | [Pinned Cryptol predicate](https://raw.githubusercontent.com/aws/s2n-tls/d695290a8e4c7107bb2c5afa359a46c0624a1c50/tests/saw/spec/handshake/rfc_handshake_tls13.cry) and [SAW invocation](https://raw.githubusercontent.com/aws/s2n-tls/d695290a8e4c7107bb2c5afa359a46c0624a1c50/tests/saw/spec/handshake/handshake.saw) | Confirmed with qualification: the predicate is parameterized by finite length; the SAW script supplies `{16}`. This is not unbounded refinement or a TLS cryptographic-security proof. |

Totals: 39 independently fetched claim checks, comprising 32 confirmed, one wrong strong interpretation, and six unverifiable. The wrong interpretation in S32 is explicitly not an accusation that the corpus fabricated the verbatim theorem. Multiple fetches of the same claim were not counted as additional checks.

## Findings

### F-001: Inadmissible excerpts still support material evidence

- **Severity:** blocker.
- **Location:** REF-C08 §§9,13,14; REF-C09 §§8,9,12–14; REF-L08 §§1,13,14. Also recheck research/07 after its in-progress rewrite.
- **Evidence:** C08 S6/S8 and C09 S5/S6 are labelled `ABSTRACT (search excerpt)` but are used for scope, independent evidence, and conflicts, rather than quarantined. L08 B4/B5 are search excerpts yet support backend maturity, toolchain and soundness-bug claims. A claimed independent result cannot be admitted simply because its title is plausible; S17 illustrates the same limitation in this review's own checking.
- **Recommended fix:** Fetch and read the relevant primary material, replace the claim with a directly inspected source, or move both the source and every dependent material assertion into unverified leads. Retain conflicts as leads if necessary, but distinguish candidate conflicts from admitted evidence. Re-run this gate after all dossiers land.
- **Final recheck:** C08/C09 now report full source reads. L08's substantive statements are independently supported by S36/S37, whose pages were read in full; update the dossier's depth labels, dates and excerpt suffix to complete the repair. L08's remaining third-party status/rename excerpts should stay explicitly provisional, not become admitted conflict evidence by proximity.

### F-002: FULL has incompatible meanings across the corpus

- **Severity:** major.
- **Location:** CONVENTIONS rule 1; original-requests interpretation of R3; research/06 introduction and A01/A03/A05/A07–A09/V08; research/07 source-depth legend.
- **Evidence:** The central rule requires the full source to be fetched and read; research/06 explicitly defines FULL as whole-document retrieval plus reading relevant sections, including cases restricted to abstracts/evaluation lines, keyword-matched lines, or a truncated document. Honest disclosure is good, but a label that means both “full text read” and “selected lines read” cannot serve as the council's binary admissibility gate.
- **Recommended fix:** Establish one central distinction between whole-source availability, inspected section scope, and claim-level support. Either obey the existing strict rule or explicitly amend the interpretation and council gate. Do not fix this merely by changing PART labels to FULL.

### F-003: Verbatim theorem text is insufficient when key predicates hide scope

- **Severity:** major.
- **Location:** REF-L01 §3; REF-L10 §3; REF-L17 §3.
- **Evidence:** Cedar's `EvaluatesTo` admits named runtime errors, independently confirmed in S32, while the displayed typecheck theorem alone can look like an error-free Boolean result. REF-L10's SLH-DSA theorem concludes `SecurityTarget core e optRand 2 1` without expanding that target into the stated quantitative bound; `ByteLaws` and `KeyDiscipline` remain critical premises. REF-L17's quorum condition is correctly exposed, and S29 shows why this discipline matters.
- **Recommended fix:** For each headline theorem, expand the result predicate and the strongest premises into plain language beside the code. Include whether error, divergence, fairness, oracle, cardinality, or correspondence conditions are permitted or assumed. Do not count opaque theorem names as substantive explanation of a security claim.
- **Final recheck:** Cedar's new predicate expansion addresses that example well. The late Clean dossier likewise makes clear that arbitrary user assumptions can render a soundness theorem vacuous; S38 confirms its conditional structure. Retain this broader semantic review for VCVio's target predicate and the assumptions in consensus/liveness capstones.

### F-004: A lexical library census is promoted into a semantic conclusion

- **Severity:** major.
- **Location:** research/06 V03; similar “for/against” tables using repository-wide theorem/sorry/axiom counts in research/01.
- **Evidence:** V03 counts lines mentioning `theorem|lemma`, then concludes concurrency/async/network libraries are “essentially specification-free.” Such a search does not inspect dependent return types, definitions of properties, instances, imported facts, program logics elsewhere, or actual theorem dependencies. Conversely, thousands of theorem-keyword hits can include trivial facts, unrelated mathematics, or conditional wrappers. S20 directly confirms that lexical zero-axiom counts do not establish axiom-free proofs.
- **Recommended fix:** Retain the measurements as lexical measurements, but replace “specification-free” with an explicitly bounded observation. Assess representative API contracts and the target theorem's dependency closure separately. Avoid placing raw counts opposite substantive limitations as if the magnitudes rebutted those limitations.
- **Final recheck:** research/06 V03 now expressly says the literal counts support no characterisation of how specified or verified a library is. That specific inference is repaired; the remaining action is to apply the same discipline consistently in summaries and side-by-side evidence tables.

### F-005: One automation percentage has an inconsistent denominator

- **Severity:** minor.
- **Location:** research/06 A03.
- **Evidence:** The entry gives 403 total unsatisfiable instances, 392 solved with proof production, and 247 reconstructed, then calls 247 “61% of the cvc5-solved instances.” But 247/392 is approximately 63.0%; 247/403 is approximately 61.3%.
- **Recommended fix:** State both numerator and denominator, and distinguish total benchmark coverage from reconstruction conditional on solver success. This arithmetic correction requires no verdict on protocol scalability.
- **Final recheck:** Corrected to 63.0% of 392 solved instances and 61.3% of 403 total instances.

### F-006: Agent-instruction files are treated as AI-authorship evidence

- **Severity:** major.
- **Location:** research/01 S4/S5/S31 and related provenance tables; research/02 S7; research/03 E34/E35.
- **Evidence:** Phrases such as “AI-agent-assisted indicated” or “AI-assistance flag” rest on AGENTS.md, CLAUDE.md, `.claude`, or `.agents`. These show agent tooling/instructions, not that any particular implementation, specification, or proof was generated by an agent. By contrast, author statements in REF-L07 and lean-zip are actual attributed claims, though still not independent provenance audits.
- **Recommended fix:** Separate “agent configuration present,” “author reports AI assistance,” and “specific provenance independently established.” Do not use the first category to discount a mathematical result or attribute proof authorship. Apply the same caution to commit-message counts and large repository size.

### F-007: Semantic analysis crosses uninspected version boundaries

- **Severity:** major.
- **Location:** REF-L07 §3.1/§6; REF-L08 §§6,13; version comparisons across research/03 and individual dossiers.
- **Evidence:** REF-L07 explains `spec` using Aeneas main but explicitly says SymCrypt's pinned older Aeneas revision was not inspected. The paper supports the intended total-correctness interpretation, but that does not establish the exact dependency version's implementation. REF-L08 compares hax main, a release, and provencrypt's separate pins, while a toolchain soundness-bug explanation depends partly on excerpts. S30 confirms that even a single artifact/report pair can name different Lean versions.
- **Recommended fix:** Anchor target-theorem semantics to its actual pinned dependency. Keep paper-intended semantics and independently inspected code semantics separate. Where a known checker bug is relevant, state affected version, exploit preconditions, whether the artifact uses the feature, and whether the target theorem was rechecked with a fixed checker; otherwise mark the impact unknown.

### F-008: The sub-claims need explicit quantifiers and scope axes

- **Severity:** major.
- **Location:** CONVENTIONS H1–H4; council-kit definitions and compound-motion round.
- **Evidence:** H1's “cannot express” can mean no protocol, not every feature, or not an adequate model of a chosen real protocol. H2 bundles safety, liveness, refinement, symbolic security and computational security under “realistic scale.” H4 switches from the owner's aspiration to a universal negative over languages, runtimes and builds. A single success, a missing feature and an unlimited-codebase requirement have different evidentiary burdens.
- **Recommended fix:** Require each ruling to specify quantifier, protocol class, property, semantics, implementation layer, assumptions, and resource envelope. Separate a claim about general logical expressiveness from a claim about a turnkey automated workflow. Do not let one existential example settle all features, or one universal “any codebase” failure settle bounded practical use.

### F-009: H5 conflates two importantly different antecedents

- **Severity:** major.
- **Location:** CONVENTIONS H5; council-kit H5 evidence and final compound motion.
- **Evidence:** “H1/H2 being true would imply H3” does not specify conjunction, disjunction, or two separate implications. Conformance testing without protocol-property proofs is relevant to a failure of H2. It is not automatically a counterexample to the stronger supposition that an adequate protocol specification cannot even be expressed in Lean, because testing against a Lean specification assumes some expressible specification.
- **Recommended fix:** Debate H1→H3 and H2→H3 separately under fixed definitions, including what a “meaningful guarantee” means. Provide a small dependency/assumption table. Preserve the owner's original compound sentence, but do not substitute an easier implication for it.

### F-010: Important implementation-refinement lines lack dossier-level treatment

- **Severity:** major.
- **Location:** research/03 E34; research/01 S15; research/04 §§7.4 and S43–S46; reference-dossier collection.
- **Evidence:** Perennial/Goose's Lean branch, Telltale's runtime correspondence claim, and Trillium/Grove/Fairneris are discussed in topic entries but not given the same theorem/premise/TCB/generalisation treatment as the selected headline references. These are particularly relevant to implementation semantics, concurrency, crash/fairness assumptions and the distinction between model proofs and source-code refinement.
- **Recommended fix:** Prioritise a Perennial/Goose dossier and an Aneris/Trillium/Grove family dossier. Add Telltale if it remains load-bearing, or explicitly demote it pending the theorem and runtime-boundary review. See Missing references for bounded reasons, not a demand for exhaustive coverage.

### F-011: Practical-scale evidence is not normalised enough for H2

- **Severity:** major.
- **Location:** research/01–06 evidence tables; council-kit H2 and scoring rubric.
- **Evidence:** Successful proofs, SMT reconstruction percentages, LOC, fuzzing budgets and AI anecdotes measure different things. The corpus usefully records several costs, but there is no comparable case table separating human specification work, proof effort, compute, elapsed time, failed attempts, maintenance, and trusted/excluded code. A small model proof, an algorithmic primitive and a deployed distributed implementation cannot be ranked by theorem count.
- **Recommended fix:** Build a compact case-comparison view from already inspected sources, marking unknown fields rather than imputing them. Require judges to state the chosen cost/scale threshold and report when the corpus cannot evaluate it. Keep general tactic benchmarks as automation evidence, not direct protocol-scale evidence.

### F-012: Reproduction evidence needs a distinct level from source inspection

- **Severity:** minor.
- **Location:** dossier §6/§10 tables generally; REF-L10's baseline-assisted counts; research/03 quote-validation account.
- **Evidence:** The dossiers mostly disclose “not built,” which is correct and useful. However, zero-token counts, baseline files, README claims that guards run on every build, and actual independently replayed proof closure remain easy to merge in later summaries. A baseline records an expected count; it is not an independently executed measurement. Static inspection can reveal S31's hole without proving the absence of all other holes.
- **Recommended fix:** Standardise four levels: author-reported, static source-inspected, independently built/rechecked, and deployed/externally replicated. Attach the level to each principal result. Preserve exact count commands/scopes if available; if not, label counts non-reproduced. A full build is not required merely to retain carefully qualified static evidence.

### F-013: The generated catalogue collapses assurance levels and mixed links

- **Severity:** major.
- **Location:** references/README.md property coverage matrix and implementation-link patterns; corresponding `_index` records.
- **Evidence:** The property legend says only yes/partial/no, so a reader can mistake a tested property for a proved property. C07 has safety marked Y although its dossier explicitly says no protocol property was proved; C08's liveness P requires the finite-trace limitation, not merely a generic partial flag. Mixed dossiers also lose relevant routes: C05's `none` hides the Tamarin-to-code bridge, while L16's `model-only` hides VeriDNS's same-language implementation. The final catalogue is more likely to be read than every underlying qualification.
- **Recommended fix:** Add assurance mode and scope to each property result, or prominently say the matrix records topics addressed rather than established guarantees and require the dossier for all guarantee claims. Record multiple implementation mechanisms and per-component scope for mixed dossiers. Rebuild from corrected metadata; do not hand-edit the generated output. A generic instruction to consult the dossier is useful but does not repair a misleading categorical cell.

## Missing references

These are priorities for strengthening the debate, not claims that the named work proves the owner's hypothesis or its negation. Clean/zkEVM, GhostBuster/VeriDNS, verified-consensus/Iris, Fiat-Crypto/s2n-tls, and CCF dossiers all landed and were reviewed; they are not omissions.

- **Perennial/Goose, including the Lean branch:** The existing E34 entry already exposes a source-language translator, crash semantics, generated axioms, an end-to-end toy network example, and explicit model-vs-Go unsoundness limitations. A dossier should trace one actual source function through translation and its theorem, and distinguish the Lean port from established Rocq results. The reference is the [Perennial repository](https://github.com/mit-pdos/perennial).
- **Aneris/Trillium/Grove/Fairneris:** research/04 gives particularly relevant implementation-refinement and concurrency/liveness contrasts, but not full dossiers. Treat each claimed refinement relation and fairness/network/crash assumption separately; do not transfer a Rocq result into Lean merely because Iris-Lean exists.
- **Telltale:** Its headline runtime-correspondence statement sits beside a “comparison-checked, not fully proved” Rust boundary and explicitly placeholder protocol families. This is a valuable adversarial case for defining meaningful implementation links, but it needs target-theorem and assumption expansion rather than reliance on inventories. The reference is the [Telltale repository](https://github.com/hxrts/telltale).
- **Verified-compilation boundary comparison, lower priority:** A short CompCert/CakeML-style comparison could clarify what a compiler theorem actually transfers and what assembler/linker/runtime assumptions remain. This need not become a general history survey; it is useful only if advocates make compiler trust a decisive distinction. Existing lean-compcert material should not be mistaken for verification of arbitrary Lean-to-native compilation.

## Unrecorded conflicts

At the initial check, the promised aggregate `research/CONFLICTS.md` was not yet present. That is a moving production dependency, not a separate blocker finding. Preserve local conflicts when aggregating them; an apparent conflict is sometimes a changed version, a different property, or a different layer rather than two incompatible facts.

| Candidate conflict or mismatch | Existing local treatment | Required handling |
|---|---|---|
| Central FULL rule versus research/06's selected-sections and truncated-source FULL entries | Disclosed locally but not reconciled centrally | Record as an evidence-policy conflict; apply F-002 rather than silently treating all labels as equivalent. |
| Cedar typecheck theorem's compact Boolean-looking conclusion versus README/topic statement allowing named runtime errors | research/02 states errors; REF-L01 now expands the predicate | Semantic scope is now explained; retain this as a model for similar hidden-predicate issues, not an outstanding contradiction. |
| Aeneas main semantics versus uninspected SymCrypt-pinned Aeneas revision | REF-L07 explicitly notes it | Preserve as a version/provenance limitation, not as demonstrated semantic disagreement. |
| Trillium's description of IronFleet node-local concurrency and pen-and-paper liveness reasoning versus the broad IronFleet safety/liveness summary | research/04 mentions the former; REF-C02 emphasises the latter | Put both into the aggregate; distinguish machine-checked layers from the meta-argument and environment assumptions. |
| Telltale axiom-free inventory, older file-table axiom counts, placeholder protocol families and top-level formal-verification claim | research/01 C7 records much of this | Carry into the central ledger; do not erase the conflict by quoting only the headline. |
| QUIC draft-29 paper attributed to Galois in topic/05 but to Crochet et al. in REF-C07 | REF-C07 §13 explicitly records the error | Correct the attribution; retain provenance of the correction instead of treating it as a substantive disagreement between researchers. |
| Trace-validation and lightweight-FM claims based partly on inadmissible search excerpts | C08/C09 source tables disclose excerpt depth | Treat as provisional leads until F-001 is fixed; a conflict ledger is not a route around evidence admission. |
| Catalogue property yes/partial cells versus dossier-specific proof/test/model qualifications | Not clearly represented by the original matrix legend | Address F-013; the summaries must not silently strengthen or conceal the underlying results. |

## Coverage by sub-claim

“For” means evidence useful to the proposition that the negative sub-claim holds under some adopted reading. “Against” means evidence useful to the opposition. These are assessments of corpus coverage, not verdicts. Both sides require explicit scope; a limitation of one tool does not establish inability of Lean in general.

| Sub-claim | For: evidence available and strength | Against: evidence available and strength | Important remaining counter-evidence or question |
|---|---|---|---|
| D | Strong diversity of usage: proof, model checking, testing, trace validation and monitoring. | Strong author caveats about their distinct guarantees. | Adopt a two-axis taxonomy: property established and implementation linkage. Avoid defining “verify” so broadly or narrowly that the debate becomes tautological. |
| H1 | Moderate evidence of missing ready-made runtime/spec interfaces, partial ports and restricted embeddings; weak evidence for impossibility. | Strong evidence of transition-system, temporal, probabilistic, adversarial and separation-logic formalisations in particular scopes. | Distinguish logical definability, usable library support and adequate modelling of actual executions. Require concrete unsupported-feature examples, not theorem counts. |
| H2 | Substantial tool limitations, reconstruction failures, assumptions, open obligations and engineering-cost anecdotes. | Substantial particular theorem/model/algorithm successes; production-adjacent primitive results and distributed-model proofs. | Comparable costs, negative attempts, maintenance experience and realistic protocol/property boundaries remain uneven. General benchmark success rates cannot fill this gap. |
| H3 | Strong evidence of trusted translators, hand-mirrored models, FFI/shim gaps, conditional premises and tested rather than proved correspondences. | Strong evidence of several meaningful but different linkage mechanisms, including extraction/translation, direct-code proof, LNSym binary semantics, CCF C++ trace replay, and empirical comparison. | Perennial/Goose and foundational refinement dossiers; pinned semantics; theorem dependency closure. Do not require an entirely assumption-free stack from only one side. |
| H4 | Strong evidence against an off-the-shelf universal-language/runtime/build workflow; many stated restrictions. | Useful source-language bridges and black-box trace/test approaches broaden applicability under weaker guarantees. | “Every codebase” versus “a route applicable after manual semantics/specification work” needs explicit quantification. Testing generality is not proof generality. |
| H5 | Material for arguing that inability to express the necessary spec would block that particular Lean-based checking route. | Good examples of checking implementations without first proving protocol-level properties, notably Cedar and conformance/trace methods. | Split the H1 and H2 antecedents. A method presupposing a Lean spec cannot by itself rebut an antecedent denying the existence of such a spec. |

The corpus is not simply one-sided. Its principal asymmetry is structural: many positive examples are narrow existential results, whereas the broad negative readings ask for universal impossibility or impracticality claims. The council should expose that mismatch rather than demand numerically equal piles of citations.

The late dossiers materially improve the range of debate evidence. REF-L16 exposes hand-transliteration, vacuity and unwitnessed-environment issues rather than presenting a uniform stack; REF-L20 separates consensus-regime premises and infrastructure ports from production implementation proof; REF-L12 separates circuit-model guarantees from exporter and cryptographic proof-system guarantees; REF-C04 distinguishes finite handshake checking from security proofs; and REF-L21 provides a concrete production Lean/C++ checking route with explicit finite-trace limitations. None should be reduced to a bare yes/no property cell.

## Dispositions

| Finding | Disposition | Action | Status |
|---|---|---|---|
| F-001 (blocker) | Accepted | C08/C09 sources re-read in full by d9. research/07 [EXT] removed and [PART] converted to [SECT] or leads (r07). L08 B4/B5 re-read in full with authors and dates corrected (d3). The checker now fails on [PART]/[EXT] outside leads. | verified closed (pass 3) |
| F-002 (major) | Accepted | CONVENTIONS now defines [FULL], [SECT] (claim-scoped) and [CODE]. ADR-0003 is amended by reference. All topic agents re-tag (Phase 1c). The council kit rule 1 is updated. | verified closed (pass 3) |
| F-003 (major) | Accepted | A "3a. Predicate semantics" section is required in every dossier. L01-L03 are done (d1), and L10 SecurityTarget expansion is assigned (d4). The council kit rule 3 requires §3a. | verified closed (pass 3) |
| F-004 (major) | Accepted | research/06 V03 restated as a lexical count with its method and limits (r06). research/01 count tables are assigned (r01). | verified closed (pass 3) |
| F-005 (minor) | Accepted | A03 corrected to 63.0% (247/392) and 61.3% (247/403). | verified closed (pass 3) |
| F-006 (major) | Accepted | Four AI-provenance categories in CONVENTIONS, plus an `ai_provenance` field in SCHEMA v2. All agents relabel. | verified closed (pass 3) |
| F-007 (major) | Accepted | L07 semantics anchored to the pinned Aeneas d71d2e3f, and checker-bug impact stated or marked unknown (d3). | verified closed (pass 3) |
| F-008 (major) | Accepted | Scope axes added to CONVENTIONS. The council kit requires scope axes on every ruling and exposes the existential/universal asymmetry. | verified closed (pass 3) |
| F-009 (major) | Accepted | H5 split into H5a (H1→H3) and H5b (H2→H3) in CONVENTIONS, the council kit and the checker. | verified closed (pass 3) |
| F-010 (major) | Accepted | New dossiers REF-L22 Perennial/Goose (d2), REF-L23 Telltale (d5), REF-C10 Aneris/Trillium/Grove/Fairneris (d6), and lower-priority REF-C11 verified-compilation boundary (d7). | verified closed (pass 3) |
| F-011 (major) | Accepted | research/CASE-COMPARISON.md with unknowns marked and no ranking (d8). Judges must state the H2 cost threshold. | verified closed (pass 3) |
| F-012 (minor) | Accepted | `evidence_level` (author-reported / static-inspected / rebuilt / deployed) in SCHEMA v2 and the catalogue. The council kit rule 3 requires it. Rebuilding is deferred to a Phase 3 CI job. | verified closed (pass 3) |
| F-013 (major) | Accepted | Property values are now proved / tested / assumed / partial / no / n/a with a legend. `impl_links` lists all mechanisms. The checker rejects legacy values. C05, C07, C08, L16 and L21 are specifically assigned. | verified closed (pass 3) |
| F-014 (major) | Accepted | Lean-SMT 247 vs 348 kept as a source conflict (research/06 A03 and C11, CF-128). Table I transcription corrected. Issue #20 tracks the external resolution. | verified closed (pass 3) |

## Disposition verification (pass 2)

This pass checked the dispositions against actual revised files at two or more locations per finding, including related conventions, source tables, predicates and metadata. Primary sources were re-fetched for several original ledger checks. Concurrent fixes were rechecked where they landed; the temporary malformed CASE-COMPARISON table was fixed before this snapshot and is not retained as a finding.

Snapshot: **11 verified closed, 2 partially closed, 0 open among F-001–F-013**. One new major finding, F-014, was identified and partly repaired during this pass, giving **11 closed, 3 partial, 0 open across all fourteen findings**. These verification statuses are separate from the orchestrator's disposition statuses above. Source disagreements may legitimately remain unresolved once they are accurately presented; that is not the same as an unaddressed review finding.

| Finding | Verified closed / partially closed / open | Evidence (file:section) | Residual risk |
|---|---|---|---|
| F-001 | Verified closed | REF-L08 §§13–15: B4/B5 FULL with correct dates, abstract-only candidate conflicts explicitly inadmissible. REF-C08 §14 and REF-C09 §14: formerly excerpt-only sources now SECT/FULL with scopes. CONVENTIONS: depth levels and rule 1. | Admission is explicit, not independent reproduction of every reported full read. Candidate conflicts based on excluded abstracts must remain leads. |
| F-002 | Verified closed | CONVENTIONS: depth levels and rule 1 now consistently allow FULL/SECT/CODE. ADR-0003: Amendment 1 defines the same standard. research/06: opening legend and A03; research/07: legend and SECT entries. | SECT is a deliberate claim-scoped amendment, not a whole-source-read claim. Generalisations beyond the named sections remain inadmissible. |
| F-003 | Verified closed | REF-L01 §3a expands allowed runtime errors and environment premises. REF-L10 §3a expands `SecurityTarget`, quantitative bound and strongest premises. REF-L12 §3a explains Soundness/Assumptions and exporter exclusions; C10 §3a separates model and implementation claims. | These are selected theorem semantics, not complete dependency-closure audits. Assumptions must still accompany each debate claim. |
| F-004 | Verified closed | research/06 V03 now expressly rejects characterising specification/verification status from lexical counts. research/01 §3 tables no longer use repo-wide counts as rebuttals. CONVENTIONS: Reproducibility records and lexical-count warning. | The inference can recur if later summaries drop these qualifications. Counts remain descriptive, not semantic assurance measurements. |
| F-005 | Verified closed | research/06 A03 separately computes 247/392 and 247/403 correctly. Its primary source introduction and §VI/Table I were independently rechecked; the conflicting source numerator is separately tracked as F-014. | Arithmetic is repaired. The source's empirical result is not a uniquely established rate merely because each division is correct. |
| F-006 | Partially closed | CONVENTIONS: controlled AI categories; research/01 opening legend and per-entry labels; catalogue AI provenance column. However research/01 S5 still says “AI-assistance flag rests on AGENTS.md and the volume of docs,” and S26 still says “flagged likely AI-generated” from an automated-update process. | Residual narrative still infers authorship from tooling despite adjacent agent-config-only labels. Delete or substantiate those inferences; no mathematical result should be discounted on that basis. |
| F-007 | Verified closed | REF-L07 §3.1, §14 C4 and §13 now inspect the actual pinned Aeneas revision. REF-L08 §§6,13 distinguish release/toolchain caveats from demonstrated exploitation. CONFLICTS CF-075 now explicitly closes the uninspected-pinned-revision gap and retains only the genuine version distinction. | Translation remains trusted. Bug impact on a particular theorem is not established; the corpus now avoids pretending otherwise. |
| F-008 | Verified closed | CONVENTIONS: Scope axes for every ruling. council-kit: Scope axes before a ruling and Cross-cutting pitfalls separate logical expressiveness, practical proof and workflow availability. | The shorthand sentence about a narrow success not refuting “universal impracticality” must be read under the axes: a qualifying counterexample does refute a genuinely universal claim. Judges must state the precise quantifier and scope. |
| F-009 | Verified closed | CONVENTIONS H5a/H5b definitions and split explanation. council-kit: separate evidence lanes, verdict round and cross-cutting pitfall. scripts/check-research: both IDs required. | Historical umbrella H5 must not re-merge the antecedents. A test requiring an expressible Lean specification does not itself rebut the inexpressibility antecedent. |
| F-010 | Verified closed | REF-L22 §§3a,5 distinguish the Lean port and mature Rocq work. REF-L23 §§3a,5 expands simulation and runtime limits. REF-C10 §§3–6 separately analyses refinement, concurrency, fairness, network and crash assumptions. REF-C11 §§3a,5 covers compilation boundaries. | The reference gaps are substantively repaired; none of the Rocq or compiler theorems should be transferred to Lean or arbitrary binaries by association. |
| F-011 | Verified closed | CASE-COMPARISON: intact comparator/Lean tables, common cost columns and Notes on use; C01–C05 and L21–L23/C10/C11 rows inspected after rewrite. council-kit: Before you start and Scope axes require case comparison and resource envelope. | Unknown costs remain unknown; the table is not a ranking. Reference-wide deployment labels do not apply to every subcomponent. |
| F-012 | Partially closed | SCHEMA.json and catalogue define/list evidence_level alongside Built. REF-L10 §§6,10 preserve baseline/static-count limits. REF-C10.json has now landed, but CASE-COMPARISON C10 rows still say “index file not yet present” at the final local check. | Refresh those case-table cells from the now-present C10 index. No new build is required. A baseline or zero textual holes still does not establish independently replayed proof closure. |
| F-013 | Verified closed | SCHEMA.json property enums and multiple impl_links; catalogue guarantee legend. REF-C07.json distinguishes testing, C08 removes liveness, L16 records same-language and mixed routes. REF-L19 §§3a,4 now removes symbolic-security claims and substantiates its secrecy subset with `shamir_secrecy` and explicit information-theoretic scope. | Leslie's computational-security partial cell must be understood only with its explicit perfect-secrecy explanation, not as a computational-hardness reduction. Any-instance and mixed-reference cells require dossier scope. C10 case-table staleness is tracked under F-012. |
| F-014 (new) | Partially closed | research/06 A03 and local Conflict C11 now expose introduction 247 versus evaluation 348; CONFLICTS CF-128 records both. But the newly expanded A03 Table I transcription shifts the baseline rows: it assigns 403/0/0 to `CVC5+simp=none` and 392/8/3 to `CVC5+Proof`. | Correct the transcription: source Table I has CVC5 403/0/0; CVC5+simp=none 392/8/3; CVC5+Proof 392/11/0; CVC5+Proof+Carcara 392/11/0; Lean-SMT 348/24/31. Keep the genuine 247-versus-348 disagreement unresolved unless clarified. |

### Independent source rechecks

- **Original S03:** The pinned Veil source still sets `veil.smt.trust` to true and states that false requests proof reconstruction; the trust caveat remains supported ([pinned Veil Base.lean](https://raw.githubusercontent.com/verse-lab/veil/57447722e7f25f06c75cff8735d10d41697afc14/Veil/Base.lean)).
- **Original S31:** The pinned Leslie `SimulationRel.soundness` body remains literally `sorry`; that observation does not establish the dependency closure of its other theorems ([pinned Refinement.lean](https://raw.githubusercontent.com/rupakm/leslie/58e58b2502b2a570e4ce4fdb6b40748c3cb30e5d/Leslie/Refinement.lean)).
- **Original S37:** The hax page still identifies October 1, 2026, the Charon+Aeneas main backend and the Lean 4.31.0 known-bug caveat; it does not establish that a particular proof exploits those bugs ([hax 0.4 announcement](https://cryspen.com/post/hax_0_4_release/)).
- **Pinned-semantics repair:** At Aeneas `d71d2e3f`, `theta` maps fail and div to False, `spec := theta`, and `dspec` maps div to True, independently supporting the repaired total-versus-partial correctness distinction ([pinned WP.lean](https://raw.githubusercontent.com/AeneasVerif/aeneas/d71d2e3f2cb763f7faf15ab606ac9cf32da8dead/backends/lean/Aeneas/Std/WP.lean)).
- **Finite-field source:** The introduction's 403/392/247 passage is genuine, but §VI-B explicitly says “LEAN-SMT can solve and reconstruct 348 instances,” corroborated by Tables I–II on the same unsatisfiable benchmark set; these are conflicting source statements, not merely solving-versus-checking quantities ([finite-field proof paper, introduction and §VI](https://hanielbarbosa.com/papers/2026fmcad-ffproofs.pdf)).

### F-014: Primary-source reconstruction counts conflict, and the repair introduces a table transcription error

- **Severity:** major.
- **Location:** research/06 A03 and local C11; CONFLICTS CF-128; source introduction versus §VI-B/Tables I–II.
- **Evidence:** The source introduction reports 247 Lean-SMT reconstructions, while the evaluation reports 348 on the same 403-unsatisfiable set; Table II also uses 348 commonly solved benchmarks ([finite-field proof paper](https://hanielbarbosa.com/papers/2026fmcad-ffproofs.pdf)). After this was reported, both values were added to the corpus, but the expanded Table I transcription assigned two rows' counts to the wrong configurations.
- **Recommended fix:** Preserve both section-specific reconstruction counts and their unresolved provenance. Correct or remove the newly mistranscribed baseline/timeout/memory-out details. Neither silently choosing the higher result nor retaining an outdated lower result is justified. This finding is about accurate debate evidence, not whether either result establishes practical protocol verification.

### Validation note

An earlier pass-2 checker run detected the concurrently missing C10 index. After that index landed, a fresh `scripts/check-research --output json` run returned zero errors and zero warnings. The remaining case-table staleness and source transcription error demonstrate why structural success does not establish semantic consistency.

Only this review file was edited during pass 2. No git command, Lean build or proof execution was run.

## Disposition verification (pass 3)

Rechecked the three remaining partial findings against the repaired files, with at least two locations per finding. All three are verified closed, bringing the final review count to **14 closed, 0 partial, 0 open findings**. This closes the review actions, not the underlying research disagreements or the hypothesis.

| Finding | Verified closed / partially closed / open | Evidence (file:section) | Residual risk |
|---|---|---|---|
| F-006 | Verified closed | research/01 S5 now states that no assistance statement was found; S22 distinguishes unknown authorship from workflow tooling; S26 calls automated updates tooling only. The requested inference-phrase searches return no matches, while per-entry provenance still distinguishes agent configuration from author reports. | Presence of agent tools does not establish authorship or weaken a theorem. The revised wording preserves that distinction. |
| F-012 | Verified closed | REF-C10.json gives `evidence_level: static-inspected` and `ai_provenance: none-found`. CASE-COMPARISON's header no longer contains the pending-index qualification; its Aneris/Trillium, Grove, Fairis and Fairneris rows now carry the matching values rather than absent-index placeholders. | Reference-wide static inspection does not mean independent rebuilding, and Fairneris's theorem remains author-reported within that inspected reference. The row's Status column explicitly retains this limitation. |
| F-014 | Verified closed | research/06 A03 now correctly transcribes all five Table I configuration rows, matching the independently inspected PDF from pass 2. Its local C11 and CONFLICTS CF-128 preserve both introduction 247 and evaluation 348 counts without choosing between them. | The paper-internal numerical disagreement remains unresolved. It is now accurately available for debate; CF-128 remaining open is consistent with this review action being closed. |

The final `scripts/check-research --output json` run returned zero errors and zero warnings. No new finding was identified in this targeted final pass, and no verdict on the owner's hypothesis is implied. Only this review file was edited; no git command or Lean build was run.
