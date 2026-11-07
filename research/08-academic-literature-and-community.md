# 08 — Academic Literature and Community Evidence: Lean 4, Protocols, Distributed Systems, Security, Implementation Verification

Phase 1 (evidence gathering only). No analysis, no conclusion. Tags record what a source says, not whether the source is correct.

- Access date for every URL below: 2026-11-07 (Saturday), unless a row says otherwise.
- Prepared for: `Profit-Rise-Consulting/code-verification-lean4`.
- Hypothesis under test (H): "Lean 4 cannot verify protocols and therefore cannot check code implementations."

## 0. Conventions

| Item | Meaning |
|---|---|
| Tag `supports` | The source states or demonstrates a limit, gap or caveat consistent with H (e.g. proof covers a model not the implementation, a language subset excluded, a trusted boundary). |
| Tag `contradicts` | The source states or demonstrates that Lean 4 (or Lean-based tooling) was used to verify a protocol and/or an implementation. |
| Tag `neutral` | Background, infrastructure, or no direct statement on H. |
| Mixed sources | Where one source contains both kinds of statements, the primary tag is the dominant claim and the other kind is quoted under "Limits stated in source" or "Evidence for". |
| P / S | Primary = the paper, repo, README, vendor/author page itself. Secondary = third-party summary, news, aggregator, blog about someone else's work. |
| Dates | Dates are as shown on the page. Where the page showed none, "n/s" (not stated). |
| Stars / last commit | GitHub REST API on 2026-11-07 (`stargazers_count`, `pushed_at`) where available. Where the API was rate-limited, star counts come from a cached GitHub page read via fetch (cache date unknown) and last-commit is "n/s". |

Source IDs are `[S##]` and are listed once in the bibliography.

---

## 1. Annotated Bibliography

### 1A. Lean 4 frameworks for protocol / transition-system verification

**[S01] Veil: A Framework for Automated and Interactive Verification of Transition Systems**
- Authors: George Pîrlea, Vladimir Gladshtein, Elad Kinsbruner, Qiyuan Zhao, Ilya Sergey. Venue: CAV 2025 (LNCS 15933, pp. 26–41), July 2025.
- URL: https://link.springer.com/chapter/10.1007/978-3-031-98682-6_2 (PDF: https://ilyasergey.net/assets/pdf/papers/veil-cav25.pdf). P.
- Abstract (factual): Open-source framework embedded in Lean 4 for machine-assisted proofs about concurrent and distributed algorithms. Gives push-button SMT-based verification (Lean-auto or Lean-SMT) for decidable first-order fragments and Lean's interactive proving when automation falls short. The paper's evaluation concludes Veil "can verify a variety of distributed protocol specifications without the need for user effort".
- Tag: contradicts (protocol/model-level verification in Lean 4 exists). Limits stated in source: see [S02], [S03].

**[S02] verse-lab/veil README and Veil/Base.lean**
- URL: https://github.com/verse-lab/veil ; https://github.com/verse-lab/veil/blob/4a17cf16/Veil/Base.lean. P.
- Facts: Veil is "a foundational framework for specifying, implementing, testing and proving safety (and, in the future, liveness) properties of state transition systems, with a focus on distributed protocols." README says liveness is future work, push-button verification applies to decidable fragments of first-order logic, and Veil 2.0 is a pre-release with "a few bugs and rough edges". Tutorial examples: Ring leader election, FloodSet crash-fault agreement. The README mentions "implementing" but, on the page read, does not distinguish protocol models from implementations.
- Tag: neutral (states both scope and limits).

**[S03] Zulip thread "Veil framework discussion thread" (stream: general), 14 Apr 2025**
- URL: https://leanprover-community.github.io/archive/stream/113488-general/topic/Veil.20framework.20discussion.20thread.html (26 messages). P.
- Facts (George Pîrlea, Shreyas Srinivas): with proof reconstruction off (the default), the SMT solver's answer is trusted, not checked. Reconstruction causes a slowdown of "more than 5x" and is "somewhat brittle". The trusted base is Lean's kernel plus cvc5 (Z3 fallback). Without a proof, Veil-generated theorems are `by sorry`, so a project is not `sorry`-free; Srinivas raised the difficulty of telling generated `sorry` from forgotten ones. The excerpt read contains no discussion of implementations vs models.
- Tag: supports (trust gap in default SMT mode), also documents mitigation (reconstruction option).

**[S04] Lessons from Building an Auto-Active Verifier in Lean**
- Authors: Pîrlea, Gladshtein, Zhao, Sergey. Venue: Dafny Workshop 2026 (co-located with POPL 2026).
- URLs: https://verse-lab.org/papers/veil-dafny26.pdf ; https://veil.dev/publications/. P.
- Facts: 18 months of continuous Veil development. States that, for SMT-based proof automation, "Veil is more than 10x slower than Ivy", with the largest case study taking 160 s vs Ivy's 50 s. Describes Veil as "a domain-specific extension of Lean" with an Ivy/RML-inspired DSL and a seamless fallback to interactive proof.
- Tag: supports (stated performance gap vs Ivy), neutral on H itself.

**[S05] Foundational Multi-Modal Program Verifiers (Loom)**
- Authors: Vladimir Gladshtein, George Pîrlea, Qiyuan Zhao, Vitaly Kurin, Ilya Sergey. Venue: POPL 2026, Proc. ACM Program. Lang. (POPL).
- URLs: https://verse-lab.org/papers/loom-popl26.pdf ; https://ilyasergey.net/publications/. P.
- Facts: Framework for embedding multi-modal program verifiers in Lean. Used to rebuild Veil as Veil 2.0 (benchmark: 17 specifications of 15 distributed protocols). Describes Veil workflow: specify, bounded model check for non-vacuity, then discover inductive invariant.
- Tag: contradicts (protocol and program verification embedded in Lean).

**[S06] Velvet: A Foundational Multi-modal Verifier for Imperative Programs in Lean**
- Authors: Gladshtein, Kurin, Feng, Kafle, Pîrlea, Zhao, Sergey. Venue: CAV 2026 (Lisbon, 26–29 Jul 2026), LNCS pp. 228–242; Distinguished Paper Award (one of 8 of 81 accepted, 311 submitted).
- URLs: https://link.springer.com/chapter/10.1007/978-3-032-32526-6_11 ; https://www.comp.nus.edu.sg/bytes/associate-professor-ilya-sergey-and-team-win-distinguished-paper-award-at-cav-2026/ (S for award) ; repo https://github.com/verse-lab/velvet. P (paper) / S (award article).
- Facts: Dafny-style verifier for imperative programs (mutable state, loops, non-determinism) embedded in Lean; combines SMT-based proofs and interactive proofs. Velvet GitHub release notes say it was rebuilt on Lean's new weakest-precondition infrastructure and `vcgen` (breaking release).
- Tag: contradicts (imperative-program verification in Lean). Note: this verifies programs written in Velvet's embedded language, not arbitrary external codebases.

**[S07] LeanDY: Type-Based and Trace-Based Symbolic Protocol Verification in Lean**
- Authors: Simon Jeanteur, Lorenzo Veronese, Magdalena Solitro, Matteo Maffei. Venue: arXiv 2607.03406, 3 Jul 2026; submitted to PLAS 2026 (not accepted/published at time of reading).
- URLs: https://arxiv.org/abs/2607.03406 ; repo https://github.com/SecPriv/leandy. P.
- Facts: Lean 4 library building on DY*; Dolev-Yao symbolic verification of stateful, unbounded protocols with a DSL and type system for confidentiality/well-formedness; XOR conditional secrecy. Case study: SegWit-style blockchain primitives and payment channels, "verifying punishment mechanisms and properties that depend on chain liveness". Abstract notes prior approaches are either automated with limited expressiveness or interactive with limited automation.
- Tag: contradicts (symbolic protocol verification in Lean 4).

**[S08] Computationally Sound Symbolic Cryptography in Lean**
- Authors: Stefan Dziembowski, Grzegorz Fabiański, Daniele Micciancio, Rafał Stefański. Venue: IEEE CSF 2026, pp. 620–635, DOI 10.1109/CSF68417.2026.00041 (ePrint 2025/1700).
- URLs: https://www.computer.org/csdl/proceedings-article/csf/2026/180600a620/2gSyOmrbUVW ; https://www.iacr.org/news/index.php?next=26670. P.
- Facts: Lean 4 framework translating symbolic cryptographic proofs into computationally sound ones, based on Micciancio (Eurocrypt 2010) and Li–Micciancio (CSF 2018) using co-induction.
- Tag: contradicts (cryptographic protocol reasoning formalised in Lean 4). Abstraction level: models, not code.

**[S09] Quantum Cryptography and Autoformalization in Lean (BB84)**
- arXiv 2610.17286, 12 Oct 2026. URL: https://arxiv.org/html/2610.17286. P.
- Facts: End-to-end composable security theorem for entanglement-based qubit BB84 against coherent attacks, in Lean, "written in a language that enforces local operations and classical communication to help ensure a faithful representation of the protocol"; builds on Lean's standard axioms only.
- Tag: contradicts (protocol security theorem in Lean).

**[S10] Leslie — shallow embedding of TLA in Lean 4 (rupakm/leslie)**
- URL: https://github.com/rupakm/leslie. P. Stars 15 (cached page).
- Facts: Provides refinement mappings, multi-action reasoning. Example status table on the README: TwoPhaseCommit, TicketLock, KVStore, LeaderBroadcast, FloodMin, BallotLeader, OneThirdRule complete; Paxos "spec complete, 2 sorry"; VRViewChange "1 sorry".
- Tag: contradicts (TLA-style protocol proofs in Lean) with open `sorry`s listed.

**[S11] LeanLTL: A Unifying Framework for Linear Temporal Logics in Lean (Short Paper)**
- Venue: ITP 2025, LIPIcs. arXiv 2507.01780. URLs: https://drops.dagstuhl.de/entities/document/10.4230/LIPIcs.ITP.2025.37 ; https://arxiv.org/html/2507.01780v1. P.
- Facts: Library for finite/infinite-trace LTL in Lean 4 that mixes LTL syntax with arbitrary Lean expressions.
- Tag: neutral (infrastructure for temporal properties).

**[S12] konnov/leanda — "Distributed algorithms in Lean 4"**
- URL: https://github.com/konnov/leanda. P.
- Facts: Author states the purpose is "to see how Lean's features match the features of TLA+".
- Tag: neutral.

**[S13] tla-plus-lean-workflow — "A Design-to-Runtime Verification Continuum: Connecting TLA+ Specifications, Lean 4 Interactive Proofs, Rust Synthesis, and Symbolic Runtime Monitoring"**
- URL: https://github.com/akkeshavan/tla-plus-lean-workflow. P (artifact repo; paper venue n/s).
- Facts: Six distributed-systems case studies through a five-phase pipeline: TLC/Apalache, Lean 4 functional model, Lean 4 invariant proofs (via Veil/SMT), Rust implementation (Aeneas "where applicable"), runtime monitoring.
- Tag: contradicts (multi-layer model-to-implementation pipeline), note "where applicable" qualifier in source.

### 1B. Distributed-protocol / consensus case studies in Lean 4

**[S14] The End of Consensus Research? (AI-greement / A-greement)**
- Authors: Burgelin, Guerraoui, Hong, Hu, Kermarrec, Pires, Veski, de Vos (EPFL). arXiv 2610.30114, Oct 2026.
- URLs: https://arxiv.org/html/2610.30114v1 ; https://paperswithlean.com/paper/2610.30114.html (S). P.
- Facts: Lean framework (ConsensusVerifier, about 135K lines, about 98K for the final protocol) proving linearizability for Multi-Paxos, FastPaxos, EPaxos*, SwiftPaxos, Bodega and the new A-greement. Safety only; "liveness is not currently covered". States proofs apply to the formal model "as modeled"; correspondence between model and prototype implementation must be checked manually. Bodega is proved linearizable only with early-accept disabled, and a counterexample is exhibited with it enabled. Reports four previously unreported vulnerabilities in existing protocols found via Lean formalisation.
- Tag: supports (explicit model-vs-implementation gap, safety-only) and contradicts (Lean protocol proofs at scale). Both quoted.

**[S15] raft-lean-squad (dsyme) — "FVSquad" report and blog "Lean Squad"**
- URLs: https://github.com/dsyme/raft-lean-squad ; https://github.com/dsyme/raft-lean-squad/blob/main/formal-verification/REPORT.md ; https://dsyme.net/2026/04/20/lean-squad-automated-software-verification-with-near-zero-human-labour/. P. Stars 11, last push 2026-06-03 (API).
- Facts: Target is raft-rs (a production Rust Raft implementation derived from TiKV's). Blog (34 runs, ~3 weeks): 22 targets, 443 theorems in 23 Lean files, "found no implementation bugs", top-level theorem `raftReachable_safe`. Report (129 runs): 716 theorems, 79 Lean files, 0 `sorry`, 28 correspondence targets validated by 625 `#guard` tests and Rust tests. The proofs are about Lean models with correspondence checked by tests.
- Tag: contradicts (Lean proofs for a model of a production implementation) with test-based correspondence, not a proved translation.

**[S16] PQ1 verification research note (EthereumPhone/PQ1)**
- URL: https://github.com/EthereumPhone/PQ1/blob/master/docs/verification/lean-verification-research-2026-06.md. S.
- Facts: Cites "1,220 theorems across raft-rs/quiche/PX4 for ~$7/run (~$240/project) with near-zero human labor, and found 3 real bugs".
- Tag: neutral (secondary claim; compare [S15], which reports 0 bugs for raft-rs in the blog).

**[S17] fradamt/verified-consensus**
- URL: https://github.com/fradamt/verified-consensus/blob/main/README.md. P. Stars 31, last push 2026-10-12 (API).
- Facts: Lean 4 model of a decoupled consensus protocol with machine-checked consensus guarantees. Results assume well-formed executions, unforgeable signatures, collision-free roots, bounded delivery after GST, honest committee majorities; liveness additionally needs participation and proposer recurrence. A "premise ledger" lists exact premises.
- Tag: contradicts (safety and liveness theorems for a model), with explicit assumptions.

**[S18] NyxFoundation formalisations: minimmit-fv, rlmd-ghost-fv, formal-leanSpec**
- URLs: https://github.com/NyxFoundation/minimmit-fv ; https://github.com/NyxFoundation/rlmd-ghost-fv/blob/main/docs/formalization-strategy.md ; https://github.com/NyxFoundation/formal-leanSpec. P.
- Stars/last push (API): minimmit-fv 2 / 2026-07-29; rlmd-ghost-fv 0 / 2026-07-29; formal-leanSpec 3 / 2026-07-30.
- Facts: minimmit-fv: Lean 4 formalisation "complete for all 10 statements (no sorry)". rlmd-ghost-fv: formalises 20 numbered statements from RLMD-GHOST (CSF 2024) and records technical barriers and policy decisions. formal-leanSpec: proofs about the Lean Ethereum `leanSpec` consensus specification.
- Tag: contradicts (protocol-paper statements proved in Lean).

**[S19] Etheorem + SizzLean (Lean 4 Ethereum consensus spec and SSZ library); ethresear.ch posts**
- URLs: https://ethresear.ch/t/lean4-ssz-library-formally-verified-and-easy-to-use/25988 ; https://ethresear.ch/t/formal-verification-of-execution-and-consensus-clients/25894 ; https://github.com/etheorem/etheorem. P (authors' own posts). Stars 34, last push 2026-11-03 (API).
- Facts: Executable Lean 4 implementation of consensus specs (Fulu, Gloas, Heze) passing official consensus-spec-tests; SizzLean SSZ codec with machine-checked proofs of three codec properties and 2188 in-scope `ssz_generic` vectors passing. A forum post describes the approach as "write the specs as code that runs, pass the official tests on them, and then add proofs and test generation on top".
- Tag: contradicts (executable Lean implementation with proofs) while proofs are of selected properties ("What is proved so far" is partial).

**[S20] Ethereum Foundation Q2 2026 allocation update**
- URL: https://blog.ethereum.org/en/2026/08/18/allocation-q2-26. P (EF) / S (as to grantees' work).
- Facts: Lists Verity (a Lean 4-based smart contract language with a formally verified compiler), "Snarkification" (verifies cryptographic protocols and zkVM circuits), and "Ethereum Protocol Security Agents – SPECA & LeanAgent" including "formal verification of Gasper Lean 4".
- Tag: contradicts (funded Lean-based protocol/contract verification programmes). Outcome status n/s.

**[S21] Verified zk(E)VM (Ethereum Foundation) — Overview, org, ArkLib**
- URLs: https://github.com/Verified-zkEVM/Overview ; https://verified-zkevm.org/ ; https://github.com/Verified-zkEVM. P.
- Facts: Aims for "formally verified bug-free zk(E)VMs a possibility by 2027". Tracks include a Lean backend for Sail, Lean circuit DSLs (cLean, zkLean), and EVM track (revm vs KEVM; KEVM shown equivalent to Nethermind's Lean EVM model). ArkLib stars 353 / last push 2026-11-06 (API).
- Tag: contradicts (programme goals include implementation-level checking), 2027 target not yet met.

**[S22] Informal and promotional Lean repos on consensus/DeFi** (recorded for completeness; quality varies)
- Chronos Vault: https://github.com/Chronos-Vault/chronos-vault-docs/blob/main/docs/formal-verification/FORMAL_VERIFICATION_STATUS.md ; https://dev.to/chronosvault/184-theorems-zero-sorry-how-we-formally-verified-a-multi-chain-protocol-57g9 ; https://github.com/Chronos-Vault/chronos-vault-contracts/blob/main/docs/verification/DEVELOPER_GUIDE.md ; https://github.com/Chronos-Vault/chronos-vault-contracts/blob/main/SECURITY_ARCHITECTURE.md. P (vendor).
  - Facts: Same vendor publishes inconsistent counts: "184 theorems, zero sorry", "33 smart contracts… 100+ theorems", "14 contracts, 42 theorems", "78/78" (one page also prints "778/78"), and "14/22 theorems proven". Developer guide states Lean 4 "proves abstract mathematical properties independent of implementation"; code-level checking is delegated to Certora and TLA+ for distributed behaviour.
  - Tag: supports (vendor states Lean layer is implementation-independent), with internal count discrepancies.
- escrow-budget: https://github.com/repowazdogz-droid/escrow-budget. P. Stars 0, last push 2026-07-28. Lean 4 proof (`Escrow.reachable_safe`) that spend ≤ cap for arbitrary finite replica sets under message loss, crash-free case; TLA+ and property tests are described as "separate, non-authoritative". Tag: contradicts.
- lean-verification-components: https://reservoir.lean-lang.org/@solanaqubits/lean-verification-components/ (and https://github.com/solanaqubits/leanbook). P. States Raft model scope: fixed nonempty cluster; "Dynamic membership, crash/recovery, Byzantine packet injection, and liveness are outside scope; this is not a certification of a deployed Raft implementation." Tag: supports.
- RaftFormalModel: https://github.com/nocaptheorem/RaftFormalModel. P. Lean 4 executable Raft model with Log Matching, State Machine Safety invariants. Tag: contradicts (model).
- Pythia: https://github.com/athanor-ai/pythia. P. Tactic library with `Pythia.Distributed` (11 theorems: Paxos quorum intersection, single-decree safety, Lamport/vector clocks, 2PC). Tag: neutral.
- PoL consensus tutorial (Hideaki Takahashi) https://freedium.cfd/a7b3d0fc563f (S mirror), repo https://github.com/Koukyosyumei/PoL (stars 5, last push 2025-08-26): tutorial verifying a simple consensus protocol's consistency; "detailed proofs of several lemmas" omitted. Tag: neutral.
- Reentrancy guard: arXiv 2606.01794, https://arxiv.org/abs/2606.01794 — "first machine-checked correctness proof of the OpenZeppelin reentrancy-guard pattern against a Lean 4 state-machine model of production-deployed Solidity source"; 13 theorems, zero sorry. P. Tag: contradicts (model of deployed source).
- Low-evidence-quality items noted but not relied on: freederia.com "Automated Formal Verification of Distributed Consensus Algorithms via Lean 4 and Symbolic Execution" (https://freederia.com/automated-formal-verification-of-distributed-consensus-algorithms-via-lean-4-and-symbolic-execution/), arXiv 2609.33066 (neural synthesis monograph with Lean proofs), OpenCLAW-P2P (https://ar5iv.labs.arxiv.org/html/2604.19792). Tag: neutral.

### 1C. Implementation verification: Rust → Lean (Aeneas / Charon / hax)

**[S23] Aeneas: Rust Verification by Functional Translation**
- Authors: Son Ho, Jonathan Protzenko. Venue: ICFP 2022 (arXiv 2206.07185).
- URL: https://ar5iv.labs.arxiv.org/html/2206.07185 . P.
- Facts: Functional translation eliminating memory reasoning "for a large class of Rust programs, as long as they do not rely on interior mutability or unsafe code". Handles shared, mutable, two-phase borrows and re-borrows. Charon 9.5 kLoC at the time.
- Tag: supports (stated subset limit) and contradicts (Rust code to Lean).

**[S24] Aeneas project pages and README**
- URLs: https://github.com/AeneasVerif/aeneas ; https://aeneasverif.github.io/projects/ ; https://www.sonho.fr/assets/documents/aeneas.html ; https://www.sonho.fr//2026/03/18/aeneas-overview.html . P.
- Facts: Project page: "Aeneas currently targets a safe, sequential Rust code." Son Ho's page: "A fundamental limitation is that the pure translation can only handle a subset of safe Rust: it can't be applied to unsafe code… and it can also not support (safe) concurrent code, interior mutability or I/O." Lean backend recommended. Repo stars 1170, last push 2026-11-07 (API). Charon: stars 477, last push 2026-11-06 (API).
- Tag: supports (explicit exclusion of concurrency, unsafe, I/O).

**[S25] Scaling Verification of Cryptographic Software with Aeneas (SymCrypt)**
- arXiv 2609.15648 (Sep 2026); authors include Antoine Delignat-Lavaud (per alphaXiv author page), Aymeric Fromherz, Jonathan Protzenko, Son Ho et al. (full list n/s in excerpt).
- URLs: https://arxiv.org/html/2609.15648 ; https://www.alphaxiv.org/zh-TW/@antoine-delignat-lavaud . P.
- Facts: 237 KLOC Lean development establishing "safety, panic-freedom, and functional correctness of 16.7 KLOC of Rust code" for post-quantum cipher suites (x86-64 and ARM). AI agents write proofs that the Lean kernel checks. Notes that side-channel protection in SymCrypt is by "leakage-specific fuzzing and symbolic execution on compiled binaries", i.e. outside the Lean proofs.
- Tag: contradicts (production code verified in Lean) and supports (side channels outside scope).

**[S26] SymCrypt verified-crypto branch README, Microsoft Research blogs, secondary coverage**
- URLs: https://github.com/microsoft/SymCrypt/blob/feature/verifiedcrypto/README-VERIFIEDCRYPTO.md (P) ; https://www.microsoft.com/en-us/research/blog/verifying-rust-cryptography-in-symcrypt-from-standards-to-code/ (P) ; https://www.microsoft.com/en-us/research/blog/rewriting-symcrypt-in-rust-to-modernize-microsofts-cryptographic-library/ (P) ; https://windowsforum.com/news/symcrypt-sha-3-and-ml-kem-proofs-ship-in-windows-insider-builds.437968/ (S) ; https://github.com/mjbommar/axeyum/blob/main/docs/prover-track/research/04-software-ir-verification.md (S).
- Facts: Complete proofs for Rust ML-KEM and SHA-3 "being used in insiders builds of Windows today". README table: ML-KEM Rust 2,125 cloc, spec 308, properties and proofs 37,989; a secondary summary gives totals of about 5,529 Rust LOC and about 58,377 proof LOC (ratio about 10.6:1). README states: "Formal theorems apply only in a model of the Rust code and the system that uses it"; "We trust the Charon/Aeneas toolchain correctly extracts code from Rust to Lean, and the soundness of its axiomatization of the Rust standard library"; trusts rustc and its x86_64/AArch64 back-ends. Secondary coverage: intrinsics "require additional assumptions". Repo stars 897 (cached page; last commit n/s). Announced 13 Jul 2026 per windowsforum (S).
- Tag: contradicts (deployed code), with trust assumptions quoted (supports-type caveats).

**[S27] A Rust-to-Lean Verification Pipeline with AI Provers: An Experience Report**
- arXiv 2605.30106 (v2 2 Jul 2026). URLs: https://arxiv.org/html/2605.30106 ; https://huggingface.co/papers/2605.30106 . P.
- Facts: Pipeline from production Rust cryptographic code (Charon+Aeneas or hax) to Lean 4 proofs, using ArkLib and CompPoly specs and AI provers (Aristotle, Aleph). Reports which obligations AI closed vs manual work and "engineering gaps… Lean 4 toolchain drift across tools and specific Aeneas/Hax extraction limits".
- Tag: contradicts with stated extraction limits.

**[S28] Verified-zkEVM/rust-lean and DevCon talk**
- URLs: https://github.com/Verified-zkEVM/rust-lean (stars 12, last push 2026-04-18, API) ; https://cfp.devcon.org/devcon8/talk/3PC37L/ . P.
- Facts: Subdirectories translate Rust to Lean 4 via hax or Aeneas and prove them (e.g. FRI folding functions). Talk abstract describes "kernel-checked Lean 4 proofs" against ArkLib/CompPoly specs.
- Tag: contradicts.

**[S29] hax 0.4 Lean backend (Cryspen) and libcrux-iot SHA-3 case study; "strengths and limits" post**
- URLs: https://cryspen.com/post/hax_0_4_release/ ; https://cryspen.com/post/sha3-hax-0-4/ ; https://cryspen.com/post/strengths-and-limitations/ ; https://hax.cryspen.com/manual/ ; https://hax.cryspen.com/blog/2026/01/19/verifying-a-real-world-rust-crate/ ; repo https://github.com/cryspen/hax (stars 488, cached). P.
- Facts: hax 0.4 uses Charon+Aeneas as its Lean backend; supports "functions returning mutable borrows". SHA-3 case: a Rust developer without Lean knowledge reviews specs, generates Lean, lets an AI agent fill proofs. Cryspen post: libcrux guarantees "only extend to the verified modules"; compiler intrinsics and the Rust compiler are unverified; hand-written unverified Rust APIs exist; Cryspen is "experimenting with using Lean for some of these proofs instead of F*".
- Tag: contradicts and supports (scope boundary stated).

**[S30] A case study with Aeneas and jxl-rs (Protzenko, 5 May 2026)**
- URL: https://jonathan.protzenko.fr/2026/05/05/jxl-rs.html . P.
- Facts: Proves panic-freedom (not full functional correctness) for selected JPEG-XL decoder functions. "Aeneas has limited support for unsafe code; unchecked accesses are treated like regular accesses". External `read_u64` needed a trusted hand-written Lean model. Missing third-party functions become axioms. Verification "explicitly partial"; `read` does not preserve the invariant after end of stream.
- Tag: supports (partial, trusted models) and contradicts (proofs on real code).

**[S31] Runtime Verification: Rust to Lean pipeline; Linux Binder; kernel-rust-verification-spike**
- URLs: https://runtimeverification.com/blog/from-rust-source-code-to-mathematical-proof ; https://runtimeverification.com/blog/verifying-linux-rust-code-from-binder-to-lean4 ; https://github.com/runtimeverification/kernel-rust-verification-spike/blob/main/AENEAS-REPORT.md . P.
- Facts: Binder blog: Charon extracted unsafe/union/MaybeUninit code "with zero changes", and where the toolchain lacks support "we re-model the affected fragment and pin the re-model to the original with compile-time assertions". Spike report: "Aeneas does not support union types… translation aborts"; union-free validators translate, with no-panic proved for three.
- Tag: supports (union/unsafe gaps) and contradicts (partial extraction and proofs).

**[S32] Solving VeriContest with a Lean-Backed Rust Verifier (Rust-Prover)**
- arXiv 2610.03994. URL: https://arxiv.org/html/2610.03994 . P.
- Facts: Verus-style Rust specs restated and translated into Lean; agents prove theorems with the Lean kernel as final check; uses forks of Charon/Aeneas.
- Tag: contradicts.

**[S33] Other Aeneas-ecosystem sources**
- zksecurity/aeneas-pp README https://github.com/zksecurity/aeneas-pp/blob/main/README.md (P, fork).
- e6qu/rust-lean-aeneas tutorials (11 tutorials; 10/11 translated; "ALL concurrency is unsupported by Aeneas"; "Functional Core, Imperative Shell"; tutorial 11: ~500 lines verified core, ~300 lines I/O shell "No (trusted)"): https://github.com/e6qu/rust-lean-aeneas/blob/main/WHAT_WE_DID.md ; https://github.com/e6qu/rust-lean-aeneas/blob/main/tutorials/11-full-integration/README.md (P, hobby/tutorial repo).
- leanprover/con-ron AENEAS_FINDINGS.md: a Lean kernel and export parser translated by Charon+Aeneas to ~73,000 lines of Lean and "partly proved to refine the Lean original"; model axiom-free: https://github.com/leanprover/con-ron/blob/master/AENEAS_FINDINGS.md (P). Stars 34, last push 2026-10-31 (API).
- REM2.0 (Refactoring and Equivalence in Rust), arXiv 2601.19207: "AENEAS is applied to safe Rust code without interior mutability or concurrency" https://ar5iv.labs.arxiv.org/html/2601.19207 (P).
- Secondary summaries of Aeneas scope: https://tamnd.github.io/brain/research/memory-management/verification/05_aeneas/ ; https://github.com/martin-ly/rust-lang/blob/main/concept/04_formal/04_model_checking/01_verification_toolchain.md (S).
- Tag: supports for the concurrency/I-O exclusion statements; contradicts for con-ron.

**[S34] Lessons Learned from a Community Effort to Verify the Rust Standard Library**
- arXiv 2510.01072v3. URL: https://arxiv.org/html/2510.01072v3 . P.
- Facts: Excerpt: "verifying the standard library must eventually include some provision for reasoning about concurrency, where appropriate."
- Tag: neutral (tool-agnostic).

### 1D. Other implementation-level Lean verification (AWS, crypto, network parsing, stdlib)

**[S35] Cedar: A New Language for Expressive, Fast, Safe, and Analyzable Authorization**
- Venue: Proc. ACM Program. Lang. (DOI 10.1145/3649835), https://dl.acm.org/doi/full/10.1145/3649835 . P.
- Facts: Cedar formalised in Lean with proofs of design properties; implementation in Rust checked against the model by "extensive differential random testing". Related AWS pages: https://aws.amazon.com/blogs/opensource/lean-into-verified-software-development/ (P); https://www.amazon.science/blog/how-we-built-cedar-with-automated-reasoning-and-differential-testing (P); https://lean-lang.org/use-cases/cedar/ (P); https://aws.amazon.com/blogs/opensource/introducing-cedar-analysis-open-source-tools-for-verifying-authorization-policies/ (P: symbolic compiler "mathematically proven" correct). A Lean-vs-Dafny history: the original model was in Dafny and was ported to Lean (https://bhaktishh.github.io/papers/popl-24-src-abs.pdf, P). Repo cedar-policy/cedar-spec: stars 222, last push 2026-11-06 (API).
- Tag: contradicts (verification-guided development in production), with the model-vs-code link established by testing, not proof.

**[S36] SampCert: Verified foundations for differential privacy**
- URLs: https://www.amazon.science/publications/verified-foundations-for-differential-privacy ; https://github.com/leanprover/SampCert (stars 107, last push 2026-09-25, API). P.
- Facts: Lean library with 12,000+ lines of proof; implementations of discrete Laplace and Gaussian samplers; "deployed and used in the AWS Clean Rooms Differential Privacy service" (README); docs page confirms use: https://docs.aws.amazon.com/zh_cn/clean-rooms/latest/userguide/differential-privacy.md (P). Venue not stated on pages read.
- Tag: contradicts (verified implementation deployed).

**[S37] LNSym: Native Code Symbolic Simulator in Lean (Armv8)**
- URLs: https://github.com/leanprover/LNSym ; https://reservoir.lean-lang.org/@leanprover/LNSym ; https://www.amazon.science/blog/how-the-lean-language-brings-math-to-coding-and-coding-to-math ; https://leodemoura.github.io/files/CAV2024.pdf . P.
- Facts: Symbolic simulator for Armv8 machine code in Lean, targeting cryptographic routines; ELF loading tests for AWS-LC; Lean specs get trust "through thorough conformance testing". Amazon says it is "currently under development".
- Tag: contradicts (machine-code verification in Lean), status under development.

**[S38] Foundations for Formally Verified Network Protocol Parsers in Lean 4: A Case Study with MQTT**
- Authors: Eduardo Sandalo Porto, Ana Cristina Vieira de Melo. Venue: SBLP 2026 (30th Simpósio Brasileiro de Linguagens de Programação), São Paulo.
- URLs: https://sol.sbc.org.br/index.php/sblp/article/view/46205 ; repo https://github.com/edusporto/lean-mqtt (stars 3, last push 2026-09-21, API). P.
- Facts: Dependently typed packet model; proof-carrying layered parser; proofs of parser roundtrip (completeness) and reconstruction (soundness) against a serializer for MQTT 5.0. Repo: "work-in-progress"; "currently only implement the parser", broker/client planned.
- Tag: contradicts (verified network-protocol parser in Lean 4) with parser-only scope.

**[S39] Lean 4 standard library HTTP (Std.Http) and Async**
- URLs: https://github.com/leanprover/lean4/blob/c9afcddf514a6772bd4d649f7cfc04eff0b9f15e/src/Std/Http.lean ; https://github.com/leanprover/lean4/pull/12151 ; https://lean-lang.org/doc/reference/latest/IO/Async/ ; https://reservoir.lean-lang.org/@algebraic-dev/Http/ ; https://leangineer.com/blog/realworld-in-lean/ ; https://daily.dev/posts/i-built-the-realworld-app-in-lean-4-x7pnmoxq7 (S). P/S.
- Facts: `Std.Http` is "a low-level HTTP/1.1 server implementation… pure, sans-I/O protocol implementation" usable with `Async`; the async runtime uses libuv for timers, sockets, signals, name resolution. A blog reports "a few thousand requests per second per process due to a runtime bug that is fixed but not yet released". Statements about proofs over Std.Http: none found in the files read.
- Tag: neutral.

**[S40] LeanServer (AfonsoBitoque) architecture**
- URL: https://github.com/AfonsoBitoque/LeanServer/blob/main/docs/ARCHITECTURE.md . P (hobby repo).
- Facts: Lists QUIC (RFC 9000) and HTTP server modules in Lean on top of a "C FFI layer" (sockets/epoll, OpenSSL). Extent of proofs: not read.
- Tag: neutral.

**[S41] Cryptography Experiments In Lean 4: SHA-3 Implementation**
- IACR ePrint 2024/1880 (Dec 2024). URL: https://eprint.iacr.org/2024/1880 . P.
- Facts: SHA-3 family implemented in Lean 4 with type classes/dependent types, with proofs of properties of the implementation.
- Tag: contradicts (implementation in Lean with proofs).

**[S42] Verifying Classic McEliece: examining the role of formal methods**
- IACR ePrint 2023/010. URL: https://eprint.iacr.org/2023/010.pdf . P.
- Facts: Mentions a Lean 4 specification of Classic McEliece "made public only after this work was concluded"; goal of that work "was to investigate the use of Lean 4 for cryptographic specification".
- Tag: neutral.

**[S43] Type-Checked Compliance: Deterministic Guardrails for Agentic Financial Systems Using Lean 4**
- arXiv 2604.01483 (Apr 2026). URL: https://arxiv.org/html/2604.01483v1 . P. Repo https://github.com/arkanemystic/lean-agent-protocol.
- Facts: Each proposed agent action is a conjecture; execution permitted only if the proof checks. Cites AWS Cedar as the "most prominent validation".
- Tag: neutral/contradicts (runtime gating using Lean as checker).

### 1E. Trust, soundness and specification-fidelity sources

**[S44] Nobody Will Read the Lean: A Case for High-Level Cryptographic Languages**
- Author: Nadim Kobeissi (Symbolic Software). IACR ePrint 2026/2441, Oct 2026. URL: https://eprint.iacr.org/2026/2441 . P.
- Facts: "What Lean can't check is whether the theorem says what you think it says." Reports a Lean library's Schnorr-signature security theorem that "passed the checker, yet as written it held for every adversary and so promised nothing". On a one-page Verifpal model "two of five verdicts would flip under other standard definitions". Argues for high-level crypto languages and publicly agreed goal definitions (e.g. forward secrecy).
- Tag: supports (specification-fidelity limit; applies to any prover, text is about Lean proofs).

**[S45] Lean kernel soundness postmortems (Jul–Aug 2026)**
- URLs: https://leodemoura.github.io/blog/2026-8-1-postmortem-for-kernel-soundness-bug-14576/ ; https://leodemoura.github.io/blog/2026-8-24-postmortem-for-the-kernel-soundness-bug-hunt/ ; https://lean-lang.org/doc/reference/latest/releases/v4.32.1/ ; https://lean-lang.org/doc/reference/latest/releases/v4.33.0/ ; https://www.openwall.com/lists/oss-security/2026/08/02/1 . P.
- Facts: Bug #14576 (nested-inductive handling) allowed a `sorry`-free "disproof" of Collatz published 25 Jul 2026; fixed the week of 27 Jul. A subsequent bug hunt with OpenAI internal models (30 Jul–20 Aug) found more, including a runtime reference-count overflow and a GMP 6.1.2 defect; fixes in v4.33.1 (21 Aug). v4.33.0 notes: some soundness bugs "survive the export format and affect proof checking via `comparator` too, unless external checkers like `nanoda` are used as well".
- Secondary commentary: Terence Tao https://terrytao.wordpress.com/2026/10/09/what-mathematicians-should-know-about-the-lean-theorem-proverquestions-of-reliability-and-ai/ ; LessWrong https://www.lesswrong.com/posts/jgmmMa7AqJNausrqx/don-t-trust-lean4-alone ; https://www.howardism.dev/articles/kernel-level-proof-auditing (relays Hales) ; https://terencekok.com/blog/when-machines-prove-what-we-cannot-follow/ (quotes de Moura: "an implementation bug, not a hole in Lean's meta-theory") (S).
- Tag: supports (trusted-base defects were found and fixed in 2026).

**[S46] Lean now ships with verified kernels / `lake check` / Y4 roadmap**
- URLs: https://leodemoura.github.io/blog/2026-10-15-lean-now-ships-with-verified-kernels/ ; https://lean-lang.org/fro/roadmap/y4-1/ . P.
- Facts: Lean v4.35.0 adds `lake check` (re-check with the official `leanchecker` in a sandbox); ships four additional kernels (nanoda, lean4lean, con-leche, con-ron); `--paranoid` rechecks with all. Y4 Part 1 roadmap (Sep 2026–Feb 2027) lists a "Trust" section and "Scalability: Mathlib and software verification". Secondary: https://00011000.com/news/lean-verified-kernels-con-leche-claude (S) says con-leche was written and proved consistent by Claude under Lean FRO supervision.
- Tag: neutral (trust-infrastructure response).

**[S47] Lean4Lean: Towards a Verified Typechecker for Lean, in Lean**
- Mario Carneiro. arXiv 2403.14064v2 (Dec 2024). URL: https://arxiv.org/html/2403.14064v2 . P.
- Facts: External checker for Lean 4 written in Lean, 20–50% slower than the C++ kernel, checks all of Mathlib.
- Tag: neutral.

**[S48] Vero: Can AI Agents Build Formally Verified Software Repositories?**
- arXiv 2608.13522. URL: https://arxiv.org/html/2608.13522 . P.
- Facts: Excerpt describes a "hollow body on a non-trivial typeclass" failure mode where an instance ignoring its arguments makes every comparison reduce to True.
- Tag: supports (specification vacuity pattern noted), neutral otherwise.

**[S49] Lean reference: `native_decide` and axioms**
- URL: https://lean-lang.org/doc/reference/latest/Axioms/ . P.
- Facts: "The `native_decide` tactic creates a bespoke axiom for each invocation. This allows each axiom to be audited."
- Tag: neutral.

### 1F. Comparison with other tools (IronFleet, Verus, TLA+)

**[S50] IronFleet: Proving Practical Distributed Systems Correct (Dafny)**
- URLs: https://cacm.acm.org/research/ironfleet/ ; https://paperswelove.org/papers/ironfleet-proving-practical-distributed-systems-co-a780f9b0/ . P/S.
- Facts: Verifies safety and liveness of implementations (IronRSL, IronKV) in Dafny using TLA-style refinement plus Hoare reasoning. Proof-to-implementation ratio 7.7:1 (5.4:1 excluding liveness); about 3.7 person-years.
- Tag: neutral (non-Lean comparator, reported as context).

**[S51] Verus: A Practical Foundation for Systems Verification (SOSP 2024) artifact**
- URL: https://verus-lang.github.io/paper-sosp24-artifact/guide.html . P.
- Facts: Includes IronKV performance comparison; claims favourable verification times against other automated tools.
- Tag: neutral.

**[S52] Tool-comparison notes (informal)**
- https://github.com/rysweet/amplihack-rs/blob/main/docs/formal-methods/verus-vs-lean-vs-tla-for-rust.md : "TLA+ verifies a design, Verus verifies the Rust implementation, and Lean 4 provides a maximal-expressiveness proof backstop for the cases the other two cannot reach." (S, informal repo doc).
- https://github.com/mizchi/formal-methods-playground/blob/main/real-world-adoption.md : Lean 4 rated "Low for bug hunting; high for final theorem" on counterexample quality (S).
- https://quint.sh/guides/formal-verification-tools : describes Veil models as "TLA+-style transition systems embedded in Lean 4" and notes proof effort "each proof has to be written down step by step" (S).
- Tag: neutral/supports (informal capability comparisons).

**[S53] AI-assisted invariant / protocol-code work adjacent to Lean**
- IC3Syn (arXiv 2605.24619, TLAPS not Lean) https://arxiv.org/html/2605.24619v1 ; Towards LM-guided TLA+ proof automation (Springer) https://link.springer.com/chapter/10.1007/978-3-032-26204-2_32 ; Testing LLM-Generated Distributed Protocol Code (ACM, covers 2PC, Ring Election, Raft) https://dl.acm.org/doi/10.1145/3819802.3820581 ; TLA-Verus arXiv 2610.14923 https://arxiv.org/html/2610.14923 . P.
- Tag: neutral (non-Lean).

### 1G. Lean FRO, industry and community announcements

**[S54] Signal Shot (Beneficial AI Foundation + Signal + Lean FRO)**
- URLs: https://www.beneficialaifoundation.org/signal-shot ; https://www.beneficialaifoundation.org/blog/signal-shot ; https://leodemoura.github.io/blog/2026-4-20-signal-shot-the-platform-is-ready/ ; https://leodemoura.github.io/static/paris2026/ ; https://www.verity.news/story/2026/baif-launches-signal-shot-to-verify-signal-protocol-using-ai (S) ; https://github.com/Beneficial-AI-Foundation/libsignal-verify . P (BAIF, Lean FRO), S (news).
- Facts: Stated goal: "verify the Signal protocol and its Rust implementation in Lean. Not the math on paper. The actual code." BAIF page: launched 20 Apr 2026 at the Software Verification in Lean workshop (Paris), prelaunch 1 Apr and 13 Apr 2026. De Moura's Paris slide says "announced April 21, 2026" (date differs from the BAIF page by one day). Components listed: Aeneas, Mathlib and CSLib, `grind`, `SymM`, AI. Plan outline: write a mathematical protocol model, prove it, then connect to Rust. Fork `libsignal-verify`: stars 11 (cached page and BAIF org listing), README says "More info to be added…", no progress detail. As of reading, no completion announcement was found.
- Tag: contradicts (stated goal and platform), outcome unknown/in progress.

**[S55] Lean FRO / CSLib**
- URLs: https://www.cslib.io/ ; https://github.com/leanprover/cslib (stars 798 per GitHub topic page, cached) ; https://leanprover-community.github.io/archive/stream/513188-CSLib/topic/Official.20Announcement.html ; CSLib papers on arXiv (2602.04846 and 2602.15078 referenced via https://www.alphaxiv.org/zh/abs/2602.15078v1) . P.
- Facts: "Mathlib for computer science": formalise CS concepts and provide infrastructure for verified software; reusable specification/labelled-transition-system interfaces and `grind`-based automation; declarations periodically checked by Lean Kernel Arena.
- Tag: neutral.

**[S56] Amazon–Lean FRO support, AWS use of Lean**
- URLs: https://www.amazon.science/news/amazon-is-investing-in-the-lean-focused-research-organization (P, 26 Jul 2026) ; https://www.itpro.com/software/development/amazon-targets-agent-safety-gains-with-investment-in-team-behind-lean-programming-language (S) ; https://leodemoura.github.io/static/aws-june-2026/ (P).
- Facts: Amazon pledges "substantial, long-term" support to the Lean FRO. Lists Lean-based verification in Bedrock AgentCore, SampCert, AWS Neuron, and "scientists using LLM-Lean combinations to prove complex distributed protocol correctness" (no paper cited on the page read).
- Tag: contradicts (stated use for distributed protocol proofs, uncited).

**[S57] &Truth funding announcement**
- URL: https://www.unite.ai/and-truth-raises-70m-seed-to-build-universal-verification-machine-for-code/ . S.
- Facts: Emerged from stealth 29 Oct 2026 with $70M seed; aims to build a "universal verification machine for code" reading code in any language. Relationship to Lean not read in the excerpt.
- Tag: neutral.

**[S58] Talks and lecture notes by Lean FRO leadership**
- URLs: https://leodemoura.github.io/static/svil26/ (SViL 2026, INRIA Paris, 20 Apr 2026) ; https://leodemoura.github.io/static/nfm2026/ ; https://leodemoura.github.io/static/floc26/ ; https://leodemoura.github.io/static/marktoberdorf2026/lecture4/index.html ; https://leodemoura.github.io/files/CAV2024.pdf ; https://leodemoura.github.io/static/LeanGoogle2025.pdf ; https://lean-lang.org/presentations/20260604_DFMD/ ; https://sebasti.a.nullri.ch/files/floc2026.pdf . P.
- Facts: Veil slides report "16 distributed protocol case studies. All 16 verified. Ivy failed on 2. 87.5% verified in under 15 seconds." (cf. Loom paper's 17 specifications of 15 protocols [S05]). Marktoberdorf lecture 4 describes a common architecture: embed the object language (Rust, a protocol language, an ISA) in Lean, give semantics, compute verification conditions, discharge them. `SymM` described as "new monadic framework for high-performance software verification". Ullrich's FLoC deck lists new `do` notation with "intrinsic verification syntax" and a verifiable Float type.
- Tag: contradicts.

**[S59] Software Verification in Lean (SViL) events**
- URLs: https://cryspen.com/post/software-verification-in-lean-2026/ (P) ; https://x.com/leanprover/status/2114845590739931589 (P, announces SViL at Stanford, Saturday 7 Nov 2026, with talks on Rust verification with Aeneas, low-level software verification and verified OS kernels; speakers include Max Tegmark, Shaowei Lin, Leonardo de Moura, Jonathan Protzenko, Andres Erbsen, Mike Dodds) . P.
- Tag: neutral.

**[S60] Veil / VERSE Lab news, POPL 2026 tutorial, lean-lang use-case page**
- URLs: https://verse-lab.org/news ; https://popl26.sigplan.org/details/POPL-2026-tutorials/9/Veil-Automated-and-Interactive-Verification-of-Transition-Systems ; https://lean-lang.org/use-cases/veil/ ; https://pirlea.net/ . P.
- Facts: Veil released 14 Apr 2025; Loom released 11 Nov 2025; blog "Proofs and Intuitions" includes "Verifying Distributed Protocols in Veil". Lean use-case page states Veil "aims to become the tool of choice for building assurance in distributed protocols".
- Tag: contradicts.

**[S61] Lean Zulip archive items touching verification or implementation**
- Stream `general`: "Veil framework discussion thread" (26 msgs, Apr 2025) [S03]; "Distributed SAT / SMT backend" (7 msgs, latest 31 Oct 2025); "Any projects rewriting Lean4 C++ parts in Rust?" (6 msgs, latest 18 Jan 2026); "lean5 - rewrite everything on rust instead of c++?" (28 msgs). URL: https://leanprover-community.github.io/archive/stream/113488-general/ . P.
- Topic "Lean in the wild": de Moura post (10 Jun 2025): "Rewriting SymCrypt in Rust… The project uses Aeneas+Lean+Mathlib." URL: https://leanprover-community.github.io/archive/stream/113488-general/topic/Lean.20in.20the.20wild.html . P.
- Topic "Lean kernel": Mario Carneiro on external typecheckers and a Lean-in-Lean verified kernel idea. URL: https://leanprover-community.github.io/archive/stream/113488-general/topic/Lean.20kernel.html . P.
- Topic "verified compiler" (stream lean4): Feb 2022 discussion on what a verified Lean 4 compiler would mean; Aug 2025 pointer to `rc-correctness` (Lean 3). URL: https://leanprover-community.github.io/archive/stream/270676-lean4/topic/verified.20compiler.html . P.
- Stream "CSLib": Official Announcement [S55].
- Not read: the private-login Zulip (live), only the public archive was accessible. Search of the public archive found no dedicated thread on protocol-implementation verification beyond the above.
- Tag: neutral.

---

## 2. Key Researchers and Groups

| Researcher / group | Affiliation (as stated) | Relevant Lean work | Sources |
|---|---|---|---|
| Ilya Sergey, George Pîrlea, Vladimir Gladshtein, Qiyuan Zhao, Elad Kinsbruner, Vitaly Kurin (VERSE Lab) | NUS (per NUS Computing news item) | Veil, Loom, Velvet | [S01][S05][S06][S60] |
| Jonathan Protzenko, Son Ho, Aymeric Fromherz, Guillaume Boisseau | Microsoft Research / Inria | Aeneas, Charon, SymCrypt proofs | [S23][S24][S25][S30] |
| Antoine Delignat-Lavaud | Microsoft (per alphaXiv author page) | SymCrypt/Aeneas paper | [S25] |
| Leonardo de Moura, Sebastian Ullrich, Joachim Breitner | Lean FRO / AWS | Lean core, kernels, `grind`, `SymM`, Signal Shot platform | [S46][S54][S58] |
| Beneficial AI Foundation (Max Tegmark) with Signal (Rolfe Schmidt) | BAIF | Signal Shot | [S54] |
| Cryspen (hax, libcrux) with CE Labs | Cryspen | hax Lean backend, libcrux-iot SHA-3 | [S29] |
| Runtime Verification | RV | Rust-to-Lean pipeline, Binder, kernel spike | [S31] |
| AWS: Emina Torlak, Kesha Hietala (Cedar), Jean-Baptiste Tristan (SampCert), Shilpi Goel (LNSym) | AWS | Cedar, SampCert, LNSym | [S35][S36][S37] |
| Matteo Maffei, Simon Jeanteur et al. (SecPriv) | TU Wien (not stated in excerpt; affiliation n/s) | LeanDY | [S07] |
| Stefan Dziembowski, Daniele Micciancio et al. | n/s | Symbolic-to-computational crypto in Lean | [S08] |
| Rachid Guerraoui, Anne-Marie Kermarrec et al. | EPFL | ConsensusVerifier | [S14] |
| Don Syme | n/s | Lean Squad / FVSquad on raft-rs | [S15] |
| Ethereum Foundation Verified zkEVM; Nyx Foundation; Nethermind (Lean EVM model) | EF | ArkLib, rust-lean, protocol formalisations | [S18][S21][S28] |
| Nadim Kobeissi | Symbolic Software | Critique of unreadable Lean crypto proofs | [S44] |
| Fabrizio Montesi et al. (CSLib) | n/s | CSLib | [S55] |
| Terence Tao, Thomas Hales (commentary) | n/s | Reliability/trust commentary | [S45] |

---

## 3. Active Repositories

Stars and last-push dates from the GitHub REST API on 2026-11-07 unless marked (c) = cached page via fetch, last commit not stated.

| Repository | Stars | Last push | Notes |
|---|---|---|---|
| leanprover/lean4 | 9,901 | 2026-11-07 | Includes `Std.Http`, `Async` |
| AeneasVerif/aeneas | 1,170 | 2026-11-07 | Rust to Lean/F*/Coq/HOL4 |
| AeneasVerif/charon | 477 | 2026-11-06 | Rust MIR to LLBC |
| leanprover/cslib | 798 (c, topic page) | n/s | CS library |
| microsoft/SymCrypt | 897 (c) | n/s | Branch `feature/verifiedcrypto` |
| cryspen/hax | 488 (c) | n/s | Lean backend via Charon+Aeneas |
| verse-lab/veil | 366 (API); README page showed 350 (c) | 2026-11-06 | Distributed protocol verifier |
| verse-lab/velvet | 134 (c) | n/s | Imperative program verifier |
| Verified-zkEVM/ArkLib | 353 | 2026-11-06 | SNARK formalisation |
| cedar-policy/cedar-spec | 222 | 2026-11-06 | Cedar Lean model |
| leanprover/SampCert | 107 | 2026-09-25 | Deployed in AWS Clean Rooms |
| etheorem/etheorem | 34 | 2026-11-03 | Ethereum consensus spec in Lean |
| leanprover/con-ron | 34 | 2026-10-31 | Alternative Lean kernel |
| fradamt/verified-consensus | 31 | 2026-10-12 | Consensus model and proofs |
| rupakm/leslie | 15 (c) | n/s | TLA embedding |
| Verified-zkEVM/rust-lean | 12 | 2026-04-18 | Rust to Lean proofs |
| dsyme/raft-lean-squad | 11 | 2026-06-03 | raft-rs proofs |
| Beneficial-AI-Foundation/libsignal-verify | 11 (c) | "Updated Nov 3, 2026" (org listing) | Signal Shot fork |
| SecPriv/leandy | 6 | 2026-07-07 | LeanDY |
| Chronos-Vault/chronos-vault-security | 6 | 2025-12-23 | Vendor repo |
| Koukyosyumei/PoL | 5 | 2025-08-26 | Tutorial |
| edusporto/lean-mqtt | 3 | 2026-09-21 | MQTT parser |
| NyxFoundation/formal-leanSpec | 3 | 2026-07-30 | |
| NyxFoundation/minimmit-fv | 2 | 2026-07-29 | |
| NyxFoundation/rlmd-ghost-fv | 0 | 2026-07-29 | |
| repowazdogz-droid/escrow-budget | 0 | 2026-07-28 | |

Not retrievable (API rate limit, no cached page read): leanprover/LNSym, leanprover/lean4lean, verse-lab/loom, leanprover-community/mathlib4, konnov/leanda, akkeshavan/tla-plus-lean-workflow, runtimeverification/kernel-rust-verification-spike, e6qu/rust-lean-aeneas, AfonsoBitoque/LeanServer. Note: the API returned "Moved Permanently" for Verified-zkEVM/VCV-io (repository renamed or moved; new location not resolved).

---

## 4. Factual Discrepancies Noted Between Sources

1. Veil case-study count: Lean FRO slides say 16 case studies, all verified, Ivy failed on 2 [S58]; Loom paper says 17 specifications of 15 protocols [S05].
2. Signal Shot date: BAIF page says launch 20 Apr 2026 [S54]; de Moura's Paris slide says announced 21 Apr 2026 [S54].
3. raft-rs: Lean Squad blog says "found no implementation bugs" for 443 theorems [S15]; the PQ1 note says "3 real bugs" across raft-rs/quiche/PX4 [S16].
4. Chronos Vault reports 184, 100+, 42, 78 (and "778/78"), and 14/22 theorems in different pages [S22].
5. Veil stars: API 366 vs README-page cache 350 [S02].

---

## 5. Open Questions

1. Does any peer-reviewed 2023–2026 paper verify a networked protocol implementation (sockets, concurrency, timers) end to end in Lean, rather than a pure core, a parser, or a model? Sources read verify models [S14][S17], parsers [S38] or pure crypto cores [S25][S26]. The Aeneas pages exclude concurrency and I/O [S24].
2. What is the status of Signal Shot deliverables (protocol model proof, Rust implementation proof)? Only launch material was found [S54].
3. What are the full results of the Std.Http / Async runtime verification story, if any? No proofs over Std.Http were found [S39].
4. Is liveness proved for any Lean protocol model beyond [S17] and LeanDY's "chain liveness" claim [S07]? Veil lists liveness as future work [S02]; ConsensusVerifier excludes it [S14].
5. How is model-to-implementation correspondence established in each project: proved (Aeneas translation, [S25]), tested (Cedar differential testing [S35], `#guard` tests [S15]), or manual review [S14]?
6. Which of the tools handle async/concurrent Rust (tokio-style) code? Aeneas says no [S24]; the hax manual and RV reports were not read for concurrency support.
7. What is the SymCrypt paper's full author list and venue (arXiv only so far)? [S25]
8. Did LeanDY or the SBLP MQTT paper get formally published after the reading date? [S07][S38]
9. Are there Lean 4 verified QUIC/TLS stacks with proofs? Only LeanServer's unproven architecture doc was found [S40].
10. Which numbers in the Lean FRO Veil slides (87.5% under 15 s) derive from the CAV paper's evaluation table (not read in full)? [S01][S58]
11. Zulip: live (login-required) Zulip search was not used; do any threads there discuss protocol-implementation verification workflows not in the public archive?
12. Mathlib4 and Lean4Lean star counts and last commits could not be retrieved; are they needed for the decision?

---

## 6. Source Ledger

- Distinct source entries above: 61 numbered entries ([S01]–[S61]), covering more than 120 distinct URLs.
- Primary vs secondary: primary-dominated; secondary sources are those explicitly tagged S (news, aggregator, third-party blog or commentary).
- Gaps: no full-text reading of the Veil CAV evaluation tables, Loom, Velvet, or the SymCrypt paper (abstract/excerpt level only); live Lean Zulip not searchable; some GitHub metrics missing due to API rate limits; the first-round search outputs were summarised from excerpts, not full documents.
