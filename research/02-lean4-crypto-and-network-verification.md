# 02 - Lean 4 in crypto and network/security verification (Phase 1b, full-read upgrade)

Header: 41 evidence entries; [FULL] 20 (16 FULL only, 4 also CODE); SECT 0; [CODE] 24 (20 CODE only); unverified leads (excluded from evidence): 11. Access date for all entries: 2026-11-07. Phase 1c re-tag of the Phase 1b file. Evidence only; no verdict is given anywhere in this file.

Tags: H1 expressiveness, H2 provability at scale, H3 spec-to-implementation link, H4 generality, H5 inference H1/H2 to H3, D definitions of "verify"/"check". [P] primary, [S] secondary. Depth [FULL] = whole source retrieved and read; SECT = named sections read (none used here); [CODE] = repository inspected statically (shallow clone, SHA recorded, counts in section 9; no builds).

Per-project dossiers by other agents live in `research/references/` and are not duplicated here: Cedar, SampCert, LNSym, Veil/Lentil/Velvet/Loom, Aeneas/SymCrypt, hax, VCVio, tls13-lean, Clean/zkEVM, LeanMqtt, Std.Http, libp2p-fv, raft-lean-squad, leanda, leslie, verified-consensus, Everest, IronFleet, Verdi, Tamarin/ProVerif, Ivy QUIC, TLA+ trace validation.

Phase 1c note: depth tags were re-assigned per CONVENTIONS (FULL, SECT, CODE); no entry here relies on a selected-sections read, so SECT is not used. Repository README reads are tagged [CODE]. Entries read only as abstract pages, search snippets or truncated text were moved to the leads section. Quotes in C1, C2, C3, V1, V3 were re-verified against fresh fetches in Phase 1c. Each entry carries a "Result level" (author-reported or static-inspected; nothing here was rebuilt or independently deployed-checked) and one AI-provenance category.

## 1. AWS: Cedar, SampCert, LNSym

Deep per-project dossiers: see `references/` (Cedar, SampCert, LNSym). This section records only the facts used for cross-project comparison.

### A1. How We Built Cedar: A Verification-Guided Approach (FSE Companion 2024)
- URL: https://arxiv.org/abs/2407.01688 (full text read via https://arxiv.org/pdf/2407.01688, 46,775 chars) | accessed 2026-11-07 | [P] | [FULL] | Tags: H1, H2, H3, H5, D
- Result level: author-reported. AI provenance: none stated in the paper.
- Definition of "verification-guided development (VGD)": "Developing a system with VGD involves writing an executable model of the system and mechanically proving properties about the model; writing production code for the system and using differential random testing (DRT) to check that the production code matches the model; and using property-based testing (PBT) to check properties of unmodeled parts of the production code."
- Implementation link: DRT (testing), not proof. Quote on why Rust is not verified directly: "tools for verifying Rust code are not (yet) up to the task, e.g., they cannot handle much of the standard library, they support only certain code idioms, they sometimes have trouble scaling, and they are limited in the properties one can specify."
- Scope quote: "Cedar's authorization engine is part of an application's trusted computing base (TCB)".
- Numbers (Table 1, LOC): Evaluator and Authorizer: Lean model 897, Lean proofs 347, Rust production 4,877. Validator: Lean model 532, Lean proofs 4,686, Rust production 6,702. Parser: no Lean model, Rust production 4,114. Total: Lean model 1,673, Lean proofs 5,714, Rust production 15,693, Rust tests 20,458, other Rust 31,391.
- Bugs: "While carrying out proofs, we found and fixed 4 bugs in Cedar's policy validator, and DRT and PBT helped us find and fix 21 additional bugs". One bug was a non-termination bug in the validator definition, found while constructing termination proofs. Table 2 lists ABAC type-directed authorizer DRT: 6 bugs; formatter round-trip: 2; validation soundness PBT: 3 ("All bugs found by the validation soundness property were found before we had completed a soundness proof").
- Test effort: each fuzz target gets 4 vCPUs and 8 GB for 6 hours daily on AWS ECS, "millions of inputs for most targets".
- Verbatim example of a proved property (Property 1): `theorem forbid_trumps_permit (request : Request) (entities : Entities) (policies : Policies) : (∃(policy : Policy), policy ∈policies ∧ policy.effect = forbid ∧ satisfied policy request entities) → (isAuthorized request entities policies).decision = deny`.
- Does not demonstrate: any proof relating the Rust code to the Lean model; the link is statistical. Cedar is an authorization policy evaluator, not a message-passing protocol.

### A2. cedar-spec repository (cedar-lean, cedar-drt)
- URL: https://github.com/cedar-policy/cedar-spec | accessed 2026-11-07 | [P] | [CODE] | Tags: H1, H2, H3, D
- Result level: static-inspected. AI provenance: none determined.
- Repo facts: commit a4c5a89cd1 (2026-11-06); `lean-toolchain` leanprover/lean4:v4.35.0; LICENSE file present (Apache-2.0 per project README, not re-verified here). 388 `.lean` files, 126,845 lines of Lean (all of the repo, including proofs and tests). Declarations counted with comments and string literals removed: `theorem`/`lemma` 2,600; `sorry` 0; `admit` 0; `axiom` 0; `native_decide` 26 (e.g. `Cedar/Thm/SymCC/Data/Ext.lean:36` `private theorem msPerDay_eq : msPerDay = (86400000 : Int64) := by native_decide`); `implemented_by` 2; `opaque` 2; `unsafe def` 9; `extern` 0.
- Verbatim from `cedar-lean/Cedar/Thm/Authorization.lean:69`: `theorem forbid_trumps_permit (request : Request) (entities : Entities) (policies : Policies) : (IsExplicitlyForbidden request entities policies) → (isAuthorized request entities policies).decision = .deny`. Line 80: `theorem allowed_only_if_explicitly_permitted ... (isAuthorized request entities policies).decision = .allow → IsExplicitlyPermitted request entities policies`. `Cedar/Thm/Typechecking.lean:47`: `theorem typecheck_is_sound (policy : Policy) (env : TypeEnv) (t : CedarType) (request : Request) (entities : Entities) : InstanceOfWellFormedEnvironment request entities env → typecheck policy env = .ok t → (∃ (b : Bool), EvaluatesTo policy.toExpr request entities b)`.
- `cedar-lean/README.md` "Verified properties": basic authorization theorems (forbid trumps permit, allowed only if explicitly permitted, order and duplicate independence); sound policy slicing; sound typechecking ("either evaluation returns a value of that type, or it returns an error of type `entityDoesNotExist`, `extensionError`, or `arithBoundsError`"); sound level-based slicing; "Sound and complete symbolic compilation"; "Sound and complete verification ... (no false negatives) and complete (no false positives)".
- Implementation link in the repo: `cedar-drt/` lists 18+ fuzz targets, each labelled "DRT" (differential random testing of Rust against the Lean model) or "PBT"; `cedar-lean-ffi/` calls the Lean model from Rust. No theorem in the repo mentions the Rust source.

### A3. Lean FRO use-case page: Cedar
- URL: https://lean-lang.org/use-cases/cedar/ | accessed 2026-11-07 | [S] (Lean FRO summary of AWS work) | [FULL] | Tags: H3, D
- Result level: author-reported (vendor summary). AI provenance: none stated.
- Quotes: "These Lean models are approximately 10 times smaller than the production code"; "AWS generates millions of random inputs and verifies that both the Lean model and Rust implementation produce identical outputs. Lean's fast execution time of only 5 microseconds per test case, compared to Rust's 7 microseconds per test case"; "No new Cedar version is released unless its model, proofs, and differential tests are current."
- Uses the phrase "mathematical certainty through formal proofs while enabling practical validation through differential testing" for the two links.

### A5. SampCert
- URL: https://github.com/leanprover/SampCert | accessed 2026-11-07 | [P] | [CODE] | Tags: H2, H3
- Result level: static-inspected. AI provenance: none determined.
- Repo facts: commit a2c53bb5ff (2026-09-25); `lean-toolchain` leanprover/lean4:v4.34.1; LICENSE present. 108 files, 18,463 lines; theorem/lemma 557; sorry 0; admit 0; axiom 0; native_decide 0; extern 6; opaque 2; implemented_by 0.
- README: "A verified implementation using Lean and Mathlib of randomized algorithms including the discrete Gaussian sampler for differential privacy, key results in zero concentrated differential privacy" and "SampCert is deployed and used in the AWS Clean Rooms Differential Privacy service." `SampCert.lean:21` states `theorem combineConcentrated [dpn : DPNoise (zCDPSystem (T := ℕ))] ...`; line 27 `theorem combinePure [dpn : DPNoise (PureDPSystem (T := ℕ))] ...`.
- A probabilistic (distribution-level) proof in Lean of a deployed component. It is not a communication protocol. The repo has `ffi.cpp` and `FastExtract.lean` for extraction; the verified-to-deployed gap was not examined in this pass (see `references/` SampCert dossier).

### A6. LNSym
- URL: https://github.com/leanprover/LNSym | accessed 2026-11-07 | [P] | [CODE] | Tags: H3
- Result level: static-inspected. AI provenance: none determined.
- Repo facts: commit 49f8a1d9e4 (2026-11-06); `lean-toolchain` leanprover/lean4:v4.35.0; LICENSE present. 202 files, 45,232 lines; theorem/lemma 675; sorry 38 (concentrated in `Tests/Tactics/CSE.lean` 11, `Proofs/SHA512/SHA512Loop.lean` 5, `Tests/Tactics/Sym.lean` 4, `Tests/Tactics/SymBlock.lean` 3); axiom 1; native_decide 194; bv_decide/decide +kernel 70.
- README: "LNSym is a symbolic simulator for Armv8 machine-code programs." Machine-code (Arm) level; the Makefile runs "conformance testing" against hardware on Aarch64. Not a protocol tool.

## 2. Microsoft, Cryspen: Rust to Lean

Deep dossiers: `references/` (Aeneas/SymCrypt, hax).

### M1. Scaling Verification of Cryptographic Software with Aeneas, Rust, and Lean (Ho, Fournet, Protzenko et al., arXiv 2609.15648, 14 Sep 2026)
- URL: https://arxiv.org/abs/2609.15648 (PDF read: https://arxiv.org/pdf/2609.15648, 88,699 chars) | accessed 2026-11-07 | [P] | [FULL] | Tags: H2, H3, H4, D
- Result level: author-reported. AI provenance: none stated in the paper.
- Abstract quotes: "Our 237 KLOC Lean development establishes safety, panic-freedom, and functional correctness of 16.7 KLOC of Rust code supporting post-quantum cipher suites for x86-64 and ARM platforms." "We design and tune our toolchain to facilitate the use of AI. Agents autonomously write formal proofs, which are independently verified by the Lean kernel. Agents also assist in the formalization of cryptographic standards and platform-specific intrinsics, which still requires expert design and review."
- Method: "we use Charon, a driver for the Rust compiler, to extract rustc's Mid-level Intermediate Representation (MIR) into a format called LLBC; we then use Aeneas to translate LLBC into an equivalent Lean model". Verification is "extrinsic": "This extrinsic style of verification clearly separates the implementation (in Rust) and its formal development".
- Trust section: "This leaves three questions for review: is it the right formal specification, is it the right set of properties for this code, and what trust assumptions do they rely on?" Trusted: the Rust compiler and back-ends, Charon and Aeneas, the Lean kernel, "the compiled native_decide and bv_decide tactics", the intrinsics model, the Aeneas axiomatisation of parts of the Rust standard library. "we verify only functional correctness, not leakage resistance."
- AI numbers: "agents generated ~5.1k theorems for a total of 125 KLOC of proofs". Figure 6 (7 hard proofs from HKDF, ML-KEM, SHA-3): cost $38.7, $39.5, $15.4 per task for Opus 4.8, GPT-5.6 Sol, Opus 5; success 6/7, 6/7, 7/7; wall time 83.5, 44.4, 29.4 min. "Models still try to 'cheat' with unsound assumptions or weakened goals, so careful human review of theorem statements remains essential."
- Performance: SymCrypt Rust SHA-3 8% faster than C; SHA-2 about 40% slower than SymCrypt C; ML-KEM with AVX2 1.3 to 1.4 times faster than C.
- Scope: primitives (SHA-3, ML-KEM, plus experimental FrodoKEM, ML-DSA, HPKE); the paper says that code "supports, for instance, post-quantum-secure TLS 1.3 ciphersuites", but proves nothing about TLS. "(Although verified, integrated, and tested, we do not claim our experimental code is production-ready.)"

### M2. SymCrypt feature/verifiedcrypto branch (README-VERIFIEDCRYPTO.md and Lean sources)
- URL: https://github.com/microsoft/SymCrypt/blob/feature/verifiedcrypto/README-VERIFIEDCRYPTO.md | accessed 2026-11-07 | [P] | [CODE] (README-VERIFIEDCRYPTO.md, 163 lines, plus Lean sources) | Tags: H2, H3, H4
- Result level: static-inspected. AI provenance: agent-config-present (AGENTS.md or CLAUDE.md files in the SymCrypt tree).
- Repo facts (branch feature/verifiedcrypto): commit c2e575ace0 (2026-07-12); `SymCRust/lean/lean-toolchain` leanprover/lean4:v4.31.0; LICENSE.txt present. `SymCRust/lean`: 170 files, 101,591 lines, theorem/lemma 1,835; sorry 0; admit 0; axiom 90 (34 in `Symcrust/Code/FunsExternal_Template.lean`, 21 in `FunsExternal.lean`, hardware-intrinsic axioms under `Intrinsics/Axioms/`); native_decide 64; bv_decide/decide +kernel 34. `AGENTS.md` present at repo root.
- Table in README (cloc): SHA-3/SHAKE: Rust 1,656, spec 140, proofs 15,211. ML-KEM: Rust 2,125, spec 308, proofs 37,989. Intrinsics: Rust 1,748, spec (axioms) 329, proofs 5,177. "The full formal development in Lean 4 is re-verified in about 15' by running `lake build`".
- Definition used: "Verified means that every public function is equipped with a theorem that specifies it in terms of the algorithms in the spec, that their proofs are complete and valid, and that the formal specification itself has been tested and reviewed against the standard."
- ML-KEM `VERIFIED.md` out of scope: "Performance, side-channel resistance, and constant-time properties. Cryptographic security of FIPS 203 itself (the standard is treated as the trusted reference)."

### M3. Developer-driven Rust verification using hax, Lean, and AI (Cryspen, 12 Oct 2026)
- URL: https://cryspen.com/post/sha3-hax-0-4/ | accessed 2026-11-07 | [P] (vendor blog) | [FULL] (17,187 chars) | Tags: H2, H3, D
- Result level: author-reported. AI provenance: author-reported (the post is about AI-assisted proof; basis: post text).
- Quotes: "hax 0.4 has a new Lean backend that relies on Aeneas". "What distinguishes the hax approach is that we try to model as much as possible in Rust: the specification is in Rust". Annotations such as `#[hax_lib::ensures(|_| future(digest).declassify_ref() == &pedantic_sha3::bytes::sha3_256(payload.declassify_ref())[..])]` "are the theorems we prove. They are written in Rust".
- Numbers: "The proofs are around 15,700 lines long"; "The initial proof development took around a week"; "The proofs were developed almost entirely by AI, with limited human guidance."
- Limitations quoted: "Not all top-level API functions have been verified. In particular, we did not verify the incremental API"; "secret independence is only ensured through Rust type mechanisms, not through Lean"; "Our proofs do not rely on any non-standard Lean axioms, but we'd like to also run comparator to rule out that the AI proved something other than what we stated." "hax does not currently support multi-crate extractions". Functions of 400+ lines had to be split to make proof iteration tractable.
- Cryspen also states that it "previously verified libcrux's SHA-3 using hax and F*" and says F* proofs "are much harder to maintain, even with AI" (author opinion).

### M4. provencrypt repository (libcrux-iot SHA-3 proofs)
- URL: https://github.com/cryspen/provencrypt | accessed 2026-11-07 | [P] | [CODE] | Tags: H2, H3
- Result level: static-inspected. AI provenance: author-reported (Cryspen post M3 says the proofs were AI-generated).
- Repo facts: commit 9f7380e894 (2026-10-13); toolchains leanprover/lean4:v4.31.0 (under `sha3/libcrux-iot-5df472a/proofs` and `sha3/spec/proofs/pedantic-sha3/lean`); no top-level LICENSE file found. 79 files, 29,262 lines; theorem/lemma 854; sorry 13; axiom 32; native_decide 0. Locations (grep on a shallow clone): 13 of the `sorry` are in `libcrux-iot-5df472a/proofs/LibcruxIotSha3/Extraction/ProofObligations.lean`, 2 in `Verification/ProofObligations.lean`, 1 each in `Sponge/Keccak.lean` and `Sponge/Shake.lean`; the 32 `axiom` lines are in the hax-generated `FunsExternal_Template.lean` (30) and `TypesExternal_Template.lean` (2) of the spec crate. Whether the main correctness theorem depends on these was not checked; the Cryspen post (M3) says the proofs use no non-standard axioms.
- README banner: "Do not use this crate in production." Per the Cryspen post (M3) the proofs were AI-generated (author-reported).

## 3. zkVM and circuit verification in Lean

Deep dossiers: `references/` (Clean/zkEVM). Note that two similarly named repositories exist: Verified-zkEVM/clean (circuit DSL, below) and Certora/clean (a CompCert fork; entry I3).

### Z1. Ethereum Foundation blog "sp1-fv": formal verification of the SP1 RISC-V zkVM in Lean
- URL: https://zkevm.ethereum.foundation/blog/sp1-fv | accessed 2026-11-07 | [P] (blog by the verification team; the Lean repo sp1-lean itself was not cloned) | [FULL] (35,950 chars) | Tags: H1, H2, H4, D
- Result level: author-reported. AI provenance: none stated.
- Scope quotes: "The sp1-lean proofs use four explicit axioms, declared in Assumptions." Two of the axioms concern kernel evaluation limits: "The axioms exist due to a Lean technical limitation rather than because the claim is in doubt. The kernel cannot compute the result by brute-force evaluation, so the authors assert it as an axiom instead." One is "SP1 memory protection disabled": "This axiom asserts the proofs assume mprotect is off, meaning all instructions are program-verified."
- Form of the theorems: "Each chip has a per-row soundness theorem of the single-step form ... 'if the input state matches the columns, this one instruction step is correct.'" (soundness only, one instruction step; there is no whole-execution statement.)
- Count correction by the blog: "The blog post claims 62 verified opcodes in three categories." "Accounting for all issues, 51 of the 62 claimed opcodes have complete, correct proofs: 40 ALU-related (all except SLTI), 7 control-flow (JAL and all six branches), and 4 stores (SB, SH, SW, SD)."
- Bug found: SP1's JALR omits the RISC-V `& ~1` mask: "The JALR AIR defines next_pc as the raw output of an AddOperation on op_b (rs1) and op_c(imm), with no bit-clearing, then checks that next_pc is divisible by 4." The blog states it emerged from "compliance testing and manual inspection", and that "This bug was outside the scope of the formal verification effort". RV64IM compliance suite: "62 out of 64 passes". Succinct "reported that they fixed it in SP1 v6.0".
- Also: FENCE "has never been implemented by SP1, neither in the executor nor in the constraint system."
- Not covered by these proofs: completeness of the circuit (for the instructions above), the prover, the recursion layers and the proof system; only per-chip constraint soundness.

### Z2. Verified-zkEVM/clean (Clean DSL)
- URL: https://github.com/Verified-zkEVM/clean | accessed 2026-11-07 | [P] | [CODE] (clean repository only) | Tags: H1, H2
- Result level: static-inspected. AI provenance: agent-config-present (AGENTS.md and CLAUDE.md in the repo).
- Repo facts (clean): see counts in the code-inspection table; AGENTS.md and CLAUDE.md are present (AI coding agents are in use, per the README line "Good AGENTS.md / CLAUDE.md"). The Verified-zkEVM Overview page was not read in this pass. This repository concerns circuit-level (arithmetisation) verification, not network protocol verification.

### Z3. EVMYulLean (Nethermind) and the Nethermind blog
- URLs: https://github.com/NethermindEth/EVMYulLean ; https://www.nethermind.io/blog/a-trustworthy-formal-model-of-evm-yul-in-lean | accessed 2026-11-07 | [P] | [FULL] (blog, 16,667 chars) + [CODE] | Tags: H1, H3, D
- Result level: author-reported (conformance tests); code static-inspected. AI provenance: none stated.
- Blog (6 Feb 2026) quotes: "We believe that we have the most complete formal model of the EVM to date, passing 99.99% (22,330/22,332) of these Cancun execution tests." "we took the conformance test suite as the ultimate source of truth, and predominantly used KEVM to understand how to bridge the differences in expected behavior between the test suite and the Yellow Paper." "we delegate the execution of EVM pre-compiles to external, efficient Python or C implementations via Lean's FFI interface."
- Meaning of "verified" here: the Lean artefact is an executable semantics validated by conformance testing; the blog describes it as a "foundation for future formal verification of smart contracts, execution clients, and zkVMs", that is, no proof about an execution client is claimed. (D, H3)

## 4. Game-based and computational cryptography in Lean

Deep dossiers: `references/` (VCVio).

### V1. VCVio: Verified Cryptography in Lean via Oracle Effects and Handlers (Tuma, Dao, Waters, Hicks, Hopper)
- URL: https://quangvdao.github.io/papers/authored/vcvio-verified-cryptography-lean.pdf | accessed 2026-11-07 | [P] | [FULL] (144,883 chars of extracted text) | Tags: H1, H2, D
- Result level: author-reported. AI provenance: author-reported (abstract: "Much of our development used LLM coding agents and automated proof-search systems").
- Abstract (verbatim): "We present VCVio, a foundational Lean 4 framework that closes both gaps using established ideas from programming-language theory: algebraic effects and handlers for oracles, and a modular relational program logic for tactics." Case studies: "a random-oracle commitment scheme; the Bellare-Neven forking lemma, mechanized without the rewindability axioms used in the recent EasyCrypt formalization by Firsov and Janku; and the Schnorr signature scheme establishing EUF-CMA security." "Much of our development used LLM coding agents and automated proof-search systems".
- Table 1 compares frameworks (CertiCrypt, EasyCrypt, FCF, CryptHOL, SSProve and VCVio; the text also discusses CatCrypt, whose footnote reads "As of this submission, CatCrypt's repository is not publicly available."). In Table 1 the EasyCrypt end-to-end EUF-CMA entry is "yes, via 11 axioms [FU22, FJ25]". The text describes CatCrypt as "a recent Lean development in the same tradition" that "adds native forking-lemma infrastructure but retains the heap/package model".
- Scope: primitives and signature schemes; no network-protocol or implementation link.

### V2. VCVio repository
- URL: https://github.com/Verified-zkEVM/VCVio | accessed 2026-11-07 | [P] | [CODE] | Tags: H1, H2
- Result level: static-inspected. AI provenance: agent-config-present (AGENTS.md and CLAUDE.md in the repo).
- Repo facts: see table. AGENTS.md and CLAUDE.md are present, so AI agents are used in development. Example files inspected: `Examples/ElGamal/Basic.lean`, `VCVio/CryptoFoundations/FiatShamir/Sigma/Fork.lean` (forking lemma material). The raw `sorry` count (6 outside docs) and `axiom` count are partly test fixtures and documentation; they were not individually triaged.

### V3. Game Hopping in Lean ("Hopscotch"; Dziembowski, Fabianski, Micciancio, Stefanski; arXiv 2608.06261, 6 Aug 2026)
- URL: https://arxiv.org/html/2608.06261v1 (repo https://github.com/ravst/Hopscotch) | accessed 2026-11-07 | [P] | [FULL] (88,600 chars) | Tags: H1, H2, D
- Result level: author-reported. AI provenance: author-reported (paper "Use of AI tools": Codex with GPT-5.5, DeepSeek-V4 and Aristotle by Harmonic used for Lean code; README: "being developed with assistance from the LLM-based tools Codex from OpenAI and Aristotle from Harmonic").
- Framework for oracle-indistinguishability (game-hopping) proofs in Lean 4; case studies: Encrypt-then-MAC, ElGamal, one-time secrecy implies IND-CPA, GGM (PRG to PRF).
- Scope quote: "similarly to most other tools, HOPSCOTCH does not verify the polynomial-time complexity of the reductions used in the proof. Instead, we rely on the concrete security bounds, or on verifying this external condition by the user." The hybrid-argument proofs are described as "semi-formal ... based on external polynomial-time assumptions verified manually by the reader".
- AI use: a section "Use of AI tools" states AI-assisted tools were used for the artifact and the paper; the repo README names Codex.

### V4. Computationally sound symbolic cryptography in Lean (Dziembowski, Fabianski, Micciancio, Stefanski; ePrint 2025/1700)
- URL: https://eprint.iacr.org/2025/1700.pdf (repo https://github.com/ravst/SymbolicCryptographyLean) | accessed 2026-11-07 | [P] | [FULL] (94,433 chars) | Tags: H1, H2, D
- Result level: author-reported. AI provenance: none stated in the text read.
- Abstract: "We present a formally-verified (in Lean 4) framework for translating symbolic cryptographic proofs into the computationally-sound ones." It formalises (1) the symbolic framework, (2) the computationally sound framework, (3) the translation between them (Micciancio, Eurocrypt 2010; Li and Micciancio, CSF 2018), and (4) an extended example of circuit garbling.
- Scope quote: "Since Lean currently misses a library for reasoning about the running time of algorithms, we use an axiomatic approach to model polynomial time. Up to our knowledge, this is handled similarly in several other tools popular tools, such as, e.g, Easycrypt, which simply assumes that every code written by the user works in polynomial time." The soundness theorem holds "as long as the underlying cryptographic primitives are secure".

### V5. Bit security of hash-based signatures in Lean4 (Kudinov, Wagner, Wambsgans; ePrint 2026/2626)
- URL: https://eprint.iacr.asia/2026/2626.pdf (repo https://github.com/b-wagn/hashbased-bitsecurity-paper-lean, commit daba20d) | accessed 2026-11-07 | [P] | [FULL] (76,921 chars) | Tags: H2
- Result level: author-reported. AI provenance: none determined.
- Abstract: "Under most of the accounting modes we consider, we get 127 bit security for strong unforgeability. However, under the most conservative convention that counts only the adversary's direct random oracle queries, we only get 122 bits of security. We prove these bounds in the random oracle model without computational hardness assumptions and give matching attacks." Built on VCVio (commit cbd4144) with Lean v4.31.0.
- Primitive-level (signature scheme) result; motivated by a Lean-checked "autoresearch" competition (sig.golf) where agents must prove bit-security claims in Lean.

### C1. CatCrypt: "From Rust to Cryptographic Security in Lean" (ePrint 2026/604)
- URL: https://eprint.iacr.asia/2026/604.pdf (full text read; 50,917 characters of extracted text) | accessed 2026-11-07 | [P] | [FULL] | Tags: H1, H2, H3, H5, D
- Result level: author-reported. AI provenance: author-reported (paper: "an experiment of what can be done by a researcher working with GenAI"; "Because it was developed with AI ..."; "GenAI was also used in the writing process").
- Claims (verbatim from abstract/body as extracted): computational-model game-based proofs in Lean covering a catalogue of constructions including TLS 1.3, SSH, WireGuard, Noise (IK/XX/KK/NK) and MLS; 138 security bounds "systematically cross-referenced ... against their published sources"; a Rust-to-Lean pipeline using the hax tool ("The translation from Rust to Lean is done using the Hax tool"). Abstract: "CatCrypt covers 172 cryptographic protocols and constructions with machine-checked security theorems in the computational model. Of these, 110 have the full Rust-to-Lean pipeline." For TLS 1.3 the paper lists "full 5-stage key schedule composition, transcript hash chain under the random oracle model, record-layer AEAD, and Finished MAC"; Noise XX is listed with the bound "3εGapDH + εHKDF + εAEAD". Core monad: `SPComp α := Heap → SDistr (α × Heap)` with `SDistr α := PMF (Option α)` (sub-probabilistic).
- Self-assessment of specification risk (verbatim): "Lean checks that our proofs are complete, but how do we know we proved the correct theorems?" Answer given: specifications grounded in IETF/NIST standards and test vectors; "a web of formally proved implications between alternative formulations of the same security notion" (e.g. IND-CPA, IND$, semantic security equivalent up to constants); definitions "cross-validated (with AI) against six textbooks".
- Provenance and availability (verbatim): "This work is an experiment of what can be done by a researcher working with GenAI ... the whole of CatCrypt was developed in a period of two months. Because it was developed with AI, we develop a new methodology to increase confidence that the specifications are correct. Moreover, we will continue to audit the code in the coming months". Section 7: "The CatCrypt sources are in a private GitHub repository, which we plan to make public as soon as we have reorganized it in a way where we can release individual protocols." Therefore the protocol-level theorems could not be inspected as Lean source in this pass. Re-check of this entry (Phase 1c): the quotes above were re-verified against a fresh fetch of the PDF text (50,917 characters).
- Does not demonstrate (as far as readable): independent replication; the connection between the Lean models and a deployed WireGuard/Noise implementation beyond the Rust-to-Lean pipeline described.

### C2. CatCrypt repository availability (cross-source fact)
- The VCVio paper (V1, Table 1 footnote, re-verified in Phase 1c) states: "As of this submission, CatCrypt's repository is not publicly available." The CatCrypt paper (C1, section 7) says its sources are in a private GitHub repository. No CatCrypt Lean source was inspected in this pass, so its protocol-level theorems are known only through the paper text. (Phase 1b wrote that Table 1 lists CatCrypt as "Open source: no"; the row text could not be re-verified, so that claim is withdrawn.) See Conflicts C-2 and Unverified leads.

### C3. Beneficial-AI-Foundation/secure-messaging
- URL: https://github.com/Beneficial-AI-Foundation/secure-messaging | accessed 2026-11-07 | [P] | [CODE] (README and repository) | Tags: H2, H3
- Result level: static-inspected. AI provenance: none stated in the README.
- README: "Formal verification of protocols for secure messaging in Lean 4, built on top of VCVio. Currently targeting cryptographic primitives and protocols from" the Double Ratchet (Alwen, Coretti, Dodis, EUROCRYPT 2019), the Triple Ratchet (Dodis et al., EUROCRYPT 2025) and a bandwidth-constrained secure-messaging comparison paper. HEAD commit "Migrate SecureMessaging and ToVCVio to the Lean module system (#404)". Repo statistics are in the code-inspection table (10 `sorry` tokens by my lexical count; not triaged). The README names targets only; it states no theorem results and no link to a Signal implementation.

### C4. NyxFoundation/libp2p-fv
- URL: https://github.com/NyxFoundation/libp2p-fv | accessed 2026-11-07 | [P] | [CODE] (README and repository; no Lean files) | Tags: H1, H3
- Result level: static-inspected. AI provenance: none determined.
- README (verbatim): "This is a research proposal, not a verified implementation or a completed end-to-end theorem" and "All obligations are **proposed**, not proved or model-checked." Intended scope: safety of Ethereum GossipSub over QUIC, "from TLS 1.3 certificate-to-Peer-ID binding through Snappy/SSZ processing to the checks that gate gossip acceptance"; it records tools as "selected" only at HEAD ("docs: select formal verification tools (#3)"). The clone contained no Lean files. Cited prior work (ACL2s GossipSub models, Floodsub refinement) is not Lean. See `references/` libp2p-fv dossier.

## 5. Network and security protocol implementations in Lean

Deep dossiers: `references/` (tls13-lean, LeanMqtt, Std.Http, libp2p-fv, Ivy QUIC, Tamarin/ProVerif, Everest). Counts for every repo below are in the code-inspection table (section 9).

### N1. tls13-lean (pb64-lean; author Bill Peterson per repo metadata)
- URL: https://github.com/pb64-lean/tls13-lean | accessed 2026-11-07 | [P] | [CODE] (README read; code counts) | Tags: H1, H2, H3, D
- Result level: static-inspected. AI provenance: none stated in the README; not determined.
- HEAD commit "Add verified mutual TLS identity and atomic credential rotation". The README's coverage summary: "The proof surface covers record framing and protection, X.509 DER retention and signature boundaries, handshake wire codecs, state-machine invariants and transitions, and key-schedule refinement through `Tls.Client.feed_keySchedule`."
- Scope limits (verbatim): "The proof boundary excludes a server `feed`-level key-schedule statement and a ..." and "No security argument is claimed for confidentiality, authentication, or the handshake as a cryptographic protocol." The README also states "the 16 `@[extern] opaque` bindings" form "the trusted C boundary" (HACL* primitives via FFI).
- Key-schedule refinement is against RFC 8446 section 7.1; branches without executable support in section 7.1 (PSK binders, early data, exporter, resumption secrets) have a specification but no refinement (as read in the README).
- AI authorship: no statement found in the README or top-level files; not determined (shallow clone, one commit visible).

### N2. LeanServer (AfonsoBitoque/LeanServer)
- URL: https://github.com/AfonsoBitoque/LeanServer | accessed 2026-11-07 | [P] | [CODE] (README, 10,107 chars, in Portuguese; code counts) | Tags: H1, H2, H3, D
- Result level: static-inspected. AI provenance: none stated in the README; not determined.
- README claims (translated): complete HTTPS server in Lean 4 with TLS 1.3, HTTP/2, HTTP/3 (QUIC), WebSocket, gRPC, "935 formal theorems verified by the compiler, zero sorry. The formal specification IS the executable code: no extraction, no spec-impl gap." Table: about 32,000 Lean lines, 81 modules, about 1,000 lines of C (FFI), 4 `partial def`, 137 executable tests, Lean v4.27.0. A "3-layer refinement chain" TLSSpec to TLSModel to ServerStep; `native_decide` is listed as a deliberate technique for finite properties. The theorem table lists categories including about 189 "Sanity" (constants validation) and about 200 "Structural" theorems, about 166 "Protocol" and about 50 "Refinement".
- Code inspection (see table): lean-toolchain v4.29.1 (not v4.27.0), 929 theorem declarations by my count, 0 sorry, 3 `admit`, 5 `axiom` lines, about 399 `native_decide`, about 77 `extern` attributes. See Conflicts C-4.
- AI authorship: not stated in the README; not determined.

### N3. Lean Squad quiche proofs (dsyme/quiche fork of Cloudflare quiche) and the Syme blog
- URLs: https://dsyme.net/2026/04/20/lean-squad-automated-software-verification-with-near-zero-human-labour/ ; https://github.com/dsyme/quiche/tree/master/formal-verification | accessed 2026-11-07 | [P] | [FULL] (blog, 29,149 chars) and [CODE] (fork) | Tags: H2, H3, D
- Result level: author-reported (blog); counts static-inspected. AI provenance: author-reported (blog: workflow "orchestrates an LLM coding agent"; AGENTS.md also present).
- Blog (20 Apr 2026): "After 85 runs, the agent has proved 518 theorems across 24 Lean 4 files with only 3 sorry remaining (two related to 8-byte varint round-trips, one for a full packet-header buffer round-trip)." Listed coverage: QUIC varint round trips, RFC 9000 A.3 packet-number decoding (`decode_pktnum_correct`), stream-ID laws, ACK range-set invariants, NewReno/CUBIC/PRR bounds, stream reassembly invariant (`insertAny_inv`), flow control (`emitN_le_maxData`). Only finding: an `Ord` antisymmetry violation in `StreamPriorityKey::cmp` that "may be intentional".
- Method statement: Lean Squad is "a declarative workflow specification that orchestrates an LLM coding agent ... as a scheduled GitHub Actions job ... Autonomously. On a schedule. With near-zero human involvement." The author's reservation (verbatim, from the closing section, which asks whether the proofs deliver the "safety" and "reliability" software needs): "These questions are unanswered by this post – and in many ways I remain skeptical."
- Link to code: Lean models in `formal-verification/` are hand-written models of selected Rust functions, with a CORRESPONDENCE.md mapping Lean to Rust; there is no extraction or mechanical proof that the Rust compiles to the Lean model. The fork contains AGENTS.md and the blog describes the proofs as produced by the agent workflow (AI-generated by the author's own description).
- Counts: blog 518 theorems (Apr 2026); repo at HEAD 2026-11 counts differ (see Conflicts C-5).

### N4. flare (ehsanmok/flare, Mojo networking library) formal/
- URL: https://github.com/ehsanmok/flare/tree/main/formal | accessed 2026-11-07 | [P] | [CODE] (formal/README.md, 17,127 chars, plus repository) | Tags: H3, H4, D
- Result level: static-inspected. AI provenance: agent-config-present (.claude and .cursor directories; README names "formal-sync (agent or human)").
- Quote: "These are Lean 4 models of flare, proofs about them, and the tooling that keeps them in sync with the Mojo code. REPORT.md describes the models and the 138 findings of the 0.12 audit, all of them fixed." Four connecting mechanisms: anchors ("tie each impl model to the Mojo symbols it transliterates, by a hash of their normalized source"), a spec lock ("fingerprints every theorem statement and spec definition ... only a human re-locks"), differential tests ("The Lean models compile to an oracle that runs against the real Mojo functions on the same inputs"), and "verified kernels" (small sans-I/O functions given a spec, proof and difftest).
- Own statement of what proofs cannot catch: "Differential tests catch what a proof can't, a model that doesn't match the code." The implementation is Mojo; the Lean side is a transliteration, so the link is by hashes and testing, not by proof.
- AI use: `.claude` and `.cursor` directories in the repo and a loop in the README that names "formal-sync (agent or human)"; so agent involvement is built into the workflow.

### N5. LeanMqtt (edusporto/lean-mqtt)
- URL: https://github.com/edusporto/lean-mqtt | accessed 2026-11-07 | [P] | [CODE] | Tags: H1, H2, H3
- Result level: static-inspected. AI provenance: none determined.
- A Lean 4 parser for MQTT 5.0 packets using dependent types so that ill-formed packets cannot be represented; theorems of round trip (parse of serialise) and soundness with respect to a serializer. Scope: packet parsing/serialisation only; no broker or session state machine. See `references/` LeanMqtt dossier. Repo counts in the table.

### N6. stdhttp-verify (eumemic)
- URL: https://github.com/eumemic/stdhttp-verify | accessed 2026-11-07 | [P] | [CODE] | Tags: H3, H4
- Result level: static-inspected. AI provenance: none determined.
- Repo description: "Differential testing and formal verification of Lean's built-in HTTP/1.1 server, `Std.Http`." Reported bug: "a valid chunked request with a chunk extension or trailer gets 400 or 200 depending on how its bytes are split across TCP reads — leanprover/lean4#15840, fix in #15841." Proofs (`spec/`, kernel-checked) are over "a patched copy of `Std.Http` (`fix/`)": the request-syntax parsers are rewritten as total, end-of-input-strict functions and proven to give [remainder of sentence not retained]. HEAD commit: "Loop: however TCP splits the input, the connection loop does the same (kernel-checked)". So proofs are about a patched Lean-language implementation, not about a C/Rust one. See `references/` Std.Http dossier.

### N7. VeriDNS (BasisResearch/VeriDNS)
- URL: https://github.com/BasisResearch/VeriDNS | accessed 2026-11-07 | [P] | [CODE] (README read) | Tags: H1, H3, D
- Result level: static-inspected. AI provenance: author-reported (README line 3: "written and verified in Lean 4 by Claude"; CLAUDE.md also present).
- README line 3: "An end-to-end DNS resolver server written and verified in Lean 4 by Claude! Verified to what spec? To the RFC itself!" Theorems read: `resolveWithIO_sound`, `ioResumeLoop_sound` (the IO loop refines a pure model), `scrubAnswer_no_foreign`. The same README states the "honesty oracle" is assumed, not proven, and the FFI transport and live network are outside the proof boundary (exercised with `dig` on UDP port 5300). CLAUDE.md present. AI-generated by its own statement.

### N8. GhostBuster (nsg-ethz/ghostbuster; GPL-3.0)
- URL: https://github.com/nsg-ethz/ghostbuster | accessed 2026-11-07 | [P] | [CODE] (README, 14k chars, plus repository) | Tags: H3, H4
- Result level: static-inspected. AI provenance: none determined.
- README: "This artifact contains the simulator, the runtime verifier, and the GNS3 testbed used to find and reproduce BGP convergence bugs in real FRR routers." Three parts: Rust simulation/verification, GNS3 testbed, and "The Lean proof" (`proof/`, "the machine-checked version of the paper's correctness argument", about 30 s check after Mathlib cache). The Lean file declares `axiom spec : State -> Msg` with the comment "We assume the spec as an axiom" and defines message-matching `InpMatchesOut`. The Lean proof is about the verifier's algorithm; FRR code is tested in GNS3, not verified in Lean.

### N9. CB-Ver: modular BGP control-plane verification (arXiv 2604.03539v3)
- URL: https://arxiv.org/html/2604.03539v3 (Lean at https://github.com/dz7903/cbgraphs-formalization, not cloned) | accessed 2026-11-07 | [P] | [FULL] (85,790 chars) | Tags: H1, H2, H3
- Result level: author-reported. AI provenance: none stated.
- Verifies BGP network configurations through an SMT encoding with a modular scheme; the soundness of the scheme (connected CB-graph theorem; k-fault-tolerance theorem) is mechanised in Lean. Verifies configurations, not router code.

### N10. qssh (Paraxiom/qssh)
- URL: https://github.com/Paraxiom/qssh | accessed 2026-11-07 | [P] | [FULL] (README) | Tags: H1, H2, D
- Result level: author-reported. AI provenance: none determined.
- README: 71 Lean lemmas on "parameter and structural conformance ... Not proofs of protocol security". Table row: "Protocol security (handshake, key derivation) | Not verified. No symbolic or computational model of the qssh handshake exists." The README states a key-exchange default bug "was found by inspection, not by a proof". Lean development archived on Zenodo.


## 6. Distributed protocols and consensus in Lean

Deep dossiers: `references/` (Veil/Lentil/Velvet/Loom, leslie, leanda, verified-consensus, raft-lean-squad, IronFleet, Verdi, TLA+ trace validation).

### D1. Veil: Foundations and Tool (CAV 2025)
- URL: https://link.springer.com/content/pdf/10.1007/978-3-031-98682-6_2 (repo https://github.com/verse-lab/veil) | accessed 2026-11-07 | [P] | [FULL] (43,180 chars) + [CODE] | Tags: H1, H2, H4, D
- Result level: author-reported (paper); counts static-inspected. AI provenance: agent-config-present (.claude skill files in the repo).
- Paper numbers: 16 case studies; 1,704 non-empty, non-comment Lean lines (about 100 per file); 85 actions; 185 invariants. SMT timeout 5 s per query (120 s for Rabia). Environment in the paper: Lean 4.16.0, cvc5 1.2.1, Z3 4.14.0. Veil verifies the four non-EPR benchmarks; the Ivy comparison (complete=fo) verifies Reliable Broadcast and Suzuki-Kasami and times out on Paxos and Vertical Paxos. Ports of the Stellar Consensus Protocol and Rabia proofs are included.
- Repo README: supports specifying, implementing, testing and proving safety; liveness is listed as future work. `.claude` skill files are present in the repo (AI tooling in use).
- Scope: transition-system models written in Veil's DSL; the "implementing/testing" claim is about executing the model, not linking to third-party implementation code.

### D3. Leslie (rupakm/leslie)
- URL: https://github.com/rupakm/leslie | accessed 2026-11-07 | [P] | [FULL] (README, 8,050 chars; not cloned, so no code counts) | Tags: H1, H2, H3
- Result level: author-reported. AI provenance: agent-config-present (.claude and AGENTS.md; repo not cloned, so seen via README/listing).
- README: "shallow embedding of TLA in Lean 4" (Lean v4.27.0) with refinement mappings, a Heard-Of round-based model and cutoff theorems. Status lines in the README: Paxos "Spec complete, 2 sorry", VRViewChange "1 sorry", OneThirdRule agreement proven for any n. A `rust/` crate (comm.rs, protocol.rs, driver.rs) and docs/rust-verification-v2.md exist; the latter was not read. `.claude` and AGENTS.md files indicate AI-assisted development.

### D4. lean-dag and the Hydrozoan paper
- URLs: https://github.com/gdanezis/lean-dag ; https://arxiv.org/html/2609.25918v1 | accessed 2026-11-07 | [P] | [FULL] (paper 131,909 chars) + [CODE] | Tags: H2, H3, D
- Result level: author-reported. AI provenance: author-reported (README "heavy LLM assistance"; paper: proofs "AI-generated and never read by humans").
- README: "heavy LLM assistance" with "only human-plus-LLM review behind it. Read critically." Theorems: `agreement` (LeanDag/FinWhale/Procedure/Protocol.lean:134), `safety` (FinWhale/Carrier.lean:356), `validated_agreement`.
- Paper: safety and liveness are machine-checked in Lean 4 "with no sorry and no axioms beyond the 3 standard ones"; about 1,000 audited lines for each of statements/definitions; "The proofs and helpers, a few thousand lines each, are AI-generated and never read by humans; the kernel checks them." Trusted-base section: a vote is a direct reference, with its equivalence to `IsVote` argued "in prose, not in Lean"; filtering of malformed blocks is assumed; "The derivation of synchrony and population from message delivery, GST and timeouts is out of scope, so the liveness theorems are exactly as strong as these two hypotheses." The implementation is a modified Mysticeti-style DAG engine run on AWS (n=50) with no formal link from the Lean model to the code.

### D5. zug-consensus-formalization (SamuelSchlesinger)
- URL: https://github.com/SamuelSchlesinger/zug-consensus-formalization | accessed 2026-11-07 | [P] | [CODE] | Tags: H2
- Result level: static-inspected. AI provenance: author-reported (README line 112: "developed with assistance from Claude (Anthropic)").
- README line 112: "developed with assistance from Claude (Anthropic)". Counts in the table.

### D6. leanda (konnov)
- URL: https://github.com/konnov/leanda | accessed 2026-11-07 | [P] | [CODE] | Tags: H2, H3
- Result level: static-inspected. AI provenance: author-reported (README line 36: Tendermint proof transferred from TLAPS "by Codex GPT 5.6").
- A collection of Lean distributed-algorithm proofs with different toolchains per sub-project (Ben-Or and tendermint-single v4.30.0; twophase v4.23.0-rc2; epfd v4.20.0-rc5). README line 36: the Tendermint proof was transferred from TLAPS "by Codex GPT 5.6". Counts in the table.

### D7. verified-consensus (fradamt)
- URL: https://github.com/fradamt/verified-consensus | accessed 2026-11-07 | [P] | [CODE] | Tags: H2
- Result level: static-inspected. AI provenance: none determined.
- The README has a "premise ledger" listing assumptions; its contents were not retained in this pass, so only the code counts (table) are recorded. See Open Questions.

## 7. Industry and programme-level sources

Deep dossiers: Cedar, SampCert, LNSym, Aeneas/SymCrypt, hax (see section 1-2 above and `references/`).

### I1. Amazon Science: Amazon invests in the Lean-focused research organization (26 Jul 2026)
- URL: https://www.amazon.science/news/amazon-is-investing-in-the-lean-focused-research-organization | accessed 2026-11-07 | [P] | [FULL] (23,563 chars) | Tags: D, H3, H4
- Result level: author-reported. AI provenance: none stated.
- Lists AWS uses of Lean: Bedrock AgentCore Policy (Cedar), SampCert (Clean Rooms differential privacy), AWS Neuron, and the sentence "One scientist recently used an LLM with Lean to prove the correctness of Amazon Aurora's segment repair protocol." No primary Aurora artefact was found (Open Questions).

### I2. Galois: automated Lean proofs (Jolt zkVM frontend)
- URL: https://www.galois.com/articles/automated-lean-proofs-for-every-type | accessed 2026-11-07 | [P] | [FULL] (12,396 chars) | Tags: H2
- Result level: author-reported. AI provenance: none stated.
- An SMT tactic is reported to have saved "over 6,800 lines of Lean code" in a project verifying the frontend of the Jolt zkVM. Tooling article; not a protocol result.

### I3. Certora/clean
- URL: https://github.com/Certora/clean | accessed 2026-11-07 | [P] | [CODE] | Tags: H3
- Result level: static-inspected. AI provenance: none determined.
- A CompCert fork with a Lean Clight backend (`export/lean-toolchain` v4.32.1; non-commercial licence). It is not the zk Clean DSL (Z2); the Phase 1 version of this file confused the two.

### I4. Least Authority on STIR/WHIR
- URL: https://leastauthority.com/blog/advancing-zkvms-with-formal-verification/ | accessed 2026-11-07 | [P] | [FULL] (3,348 chars) | Tags: H2
- Result level: author-reported. AI provenance: none stated.
- An Ethereum Foundation Verified zkEVM grant for a Lean "blueprint" for STIR and WHIR (theorem statements and proof outline; no completed proofs claimed in the post).

## 8. Conflicts between sources

Recorded without resolution. Each item lists both sides.

- C-1. Rust verification. Side 1: the Cedar paper (A1) says "tools for verifying Rust code are not (yet) up to the task" and uses differential random testing to link Lean model and Rust code. Side 2: the SymCrypt/Aeneas paper (M1, 2026) reports Lean proofs about Rust code translated by Aeneas. Dates and targets differ (2024 paper on Cedar; 2026 paper on SymCrypt); whether both statements hold for the same Rust subset is not established by these sources.
- C-2. Lean proofs of full handshake security. Side 1: CatCrypt (C1) claims machine-checked computational security theorems for TLS 1.3, SSH, WireGuard and Noise variants, while the VCVio paper (V1, Table 1) records the CatCrypt repository as "not publicly available" and the CatCrypt paper says its sources are in a private repository (so independent replication was impossible in this pass). Side 2: tls13-lean (N1) states "No security argument is claimed for confidentiality, authentication, or the handshake as a cryptographic protocol", and qssh (N10) states its handshake security is "Not verified". These are different projects with different aims; the sources do not compare them.
- C-3. Treatment of polynomial time. ePrint 2025/1700 (V4): "we use an axiomatic approach to model polynomial time". Hopscotch (V3): does not verify polynomial-time complexity and relies on concrete bounds or user-verified external conditions. CatCrypt (C1) states concrete (not asymptotic) bounds. The VCVio treatment of efficiency was not compared line by line here.
- C-4. LeanServer (N2). README: "935 formal theorems", 0 sorry, "no spec-impl gap", Lean v4.27.0. My lexical count (keyword tokens, not a proof-closure audit) at SHA 24b916aaa6: 929 theorem/lemma declarations, 0 sorry, 3 `admit`, 5 `axiom` lines, about 399 `native_decide`, 77 `extern`/`opaque` bindings with about 1,000 lines of C FFI, toolchain v4.29.1. Whether the `admit` and `axiom` lines are in proof code or in tests was not triaged.
- C-5. quiche theorem counts. Blog (N3), Apr 2026: 518 theorems, 24 files, 3 sorry. Repo at SHA 5a8195ebf9 (2026-06-03): my lexical count 1,592 theorem/lemma keyword declarations in 81 files, 0 sorry, 258 `native_decide`. The repo's CRITIQUE.md (read in part) gives yet another figure (1,348). The differences may reflect later agent runs; not verified.
- C-6. VeriDNS (N7). README: "Verified to what spec? To the RFC itself!" Same README: the "honesty oracle" is assumed and the FFI transport and live network are outside the proof.
- C-7. Veil liveness. Veil README: safety supported, liveness a future item. The SoK (arXiv 2608.21935, partially read, see leads) says that as of March 2026 Veil does not support liveness natively. The two statements agree on direction; the Leslie README (D3) lists liveness-style round-based results under a different framework. No contradiction is established, but the SoK was not fully read.
- C-8. raft-lean-squad (candidate conflict from a search snippet; repository not inspected, see leads). Snippet of its REPORT.md says all five `RaftReachable.step` hypotheses were "fully discharged"; the Phase 1 version of this file read earlier states of the same repository that some hypotheses were bundled axioms. Not read in full in this pass, so this is a candidate conflict only and is unresolved.
- C-9. "Verified" with AI-generated, unread proofs. Hydrozoan paper (D4) states safety and liveness are machine-checked and that proofs are "AI-generated and never read by humans"; the kernel is the only reviewer of proof text, and the human-audited part is the statements (about 1,000 lines). The README of the same repo says "Read critically." The sources frame this as acceptable under kernel checking; other sources in this file (qssh, tls13-lean, libp2p-fv) emphasise explicit non-claims.
- C-10. "Verified" for model versus implementation. EVMYulLean (Z3) is validated by conformance tests (22,330 of 22,332); flare (N4) links Mojo to Lean by hashes plus differential tests; Cedar (A1) uses differential random testing; SymCrypt (M1/M2) uses Aeneas translation of Rust; LeanServer (N2) claims no gap because implementation is Lean. The sources use "verify" for these different relations (see D).


## 9. Code-inspection table (CONVENTIONS rule 3)

Method: `git clone --depth 1 --filter=blob:limit=1m` into /tmp, SHA and date from the clone, then a small Python script (`cnt.py`) that strips comments and string literals and counts tokens in `.lean` files. No builds were run and the GitHub REST API was not used. Counts are approximate: "theorem" counts `theorem` and `lemma` declarations; "axiom" counts declaration lines; "extern" counts `@[extern]` attributes; counts include test, example and fixture directories (VCVio and LNSym `sorry`/`axiom` hits were not triaged between library and test code). Clones were deleted afterwards because of the disk limit, so these counts cannot be re-run from this machine without re-cloning. Cedar, SampCert, LNSym, SymCrypt and provencrypt are described in section 1-2. A repo with 0 files has no Lean source (libp2p-fv; lean4-security-proofs is a text-only tutorial).

| Repo | SHA | HEAD date | lean-toolchain | Licence file | Lean files | LOC | theorem+lemma | sorry | admit | axiom | native_decide | implemented_by | extern | opaque | unsafe |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| EVMYulLean | 047f630703 | 2025-09-24 | leanprover/lean4:v4.22.0 | license.txt | 66 | 11339 | 39 | 0 | 0 | 0 | 0 | 1 | 4 | 4 | 2 |
| Hopscotch | 2699b4feda | 2026-08-10 | leanprover/lean4:v4.28.0 | LICENSE | 70 | 12827 | 309 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| LNSym | 49f8a1d9e4 | 2026-11-06 | leanprover/lean4:v4.35.0 | LICENSE | 202 | 45232 | 675 | 38 | 0 | 1 | 194 | 1 | 0 | 0 | 0 |
| LeanServer | 24b916aaa6 | 2026-05-14 | leanprover/lean4:v4.29.1 | LICENSE | 108 | 38581 | 929 | 0 | 3 | 5 | 399 | 0 | 77 | 77 | 0 |
| SampCert | a2c53bb5ff | 2026-09-25 | leanprover/lean4:v4.34.1 | LICENSE | 108 | 18463 | 557 | 0 | 0 | 0 | 0 | 0 | 6 | 2 | 0 |
| SymCrypt | c2e575ace0 | 2026-07-12 | (not at root) | LICENSE.txt | 171 | 103618 | 1835 | 0 | 0 | 101 | 64 | 0 | 0 | 1 | 0 |
| VCVio | 0b69833ce5 | 2026-11-06 | leanprover/lean4:v4.35.0 | LICENSE | 1613 | 437887 | 15486 | 6 | 0 | 7 | 9 | 7 | 28 | 32 | 6 |
| VeriDNS | 74bb5f3f14 | 2026-07-17 | leanprover/lean4:v4.31.0-rc2 | none found | 128 | 83393 | 2866 | 0 | 0 | 0 | 0 | 0 | 14 | 14 | 0 |
| cedar-spec | a4c5a89cd1 | 2026-11-06 | leanprover/lean4:v4.35.0 | LICENSE | 388 | 126845 | 2600 | 0 | 0 | 0 | 26 | 2 | 0 | 2 | 9 |
| certora-clean | 123f7f7767 | 2026-09-08 | (not at root) | LICENSE | 72 | 27702 | 706 | 0 | 0 | 4 | 0 | 0 | 0 | 0 | 0 |
| etheorem | 417ca909aa | 2026-11-01 | leanprover/lean4:v4.34.1 | LICENSE | 369 | 61202 | 645 | 0 | 0 | 5 | 662 | 10 | 39 | 41 | 28 |
| flare | 576799df50 | 2026-11-02 | (not at root) | LICENSE | 171 | 54125 | 2695 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| formal-leanSpec | ba72845130 | 2026-07-30 | leanprover/lean4:v4.29.1 | none found | 41 | 8534 | 222 | 0 | 0 | 1 | 0 | 0 | 0 | 0 | 0 |
| ghostbuster | 7564d6e4e5 | 2026-07-30 | (not at root) | LICENSE | 1 | 309 | 20 | 0 | 0 | 1 | 0 | 0 | 0 | 0 | 0 |
| lean-dag | dcd2f81bf7 | 2026-11-03 | leanprover/lean4:v4.32.2 | LICENSE | 769 | 122799 | 3890 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| lean-grpc | 2e7712ae2f | 2026-08-12 | leanprover/lean4:v4.32.1 | LICENSE | 105 | 13418 | 103 | 0 | 0 | 0 | 71 | 0 | 11 | 13 | 0 |
| lean-mqtt | af23e7bea5 | 2026-09-21 | leanprover/lean4:v4.34.0 | LICENSE | 42 | 4751 | 150 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| lean4-security-proofs | 74114f5965 | 2026-10-23 | (not at root) | LICENSE | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| leanda | f7abaa1d04 | 2026-07-28 | (not at root) | LICENSE | 25 | 19196 | 387 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| libp2p-fv | 1f25f34852 | 2026-11-05 | (not at root) | none found | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| pg-lean | 6d1d1043cd | 2026-09-10 | leanprover/lean4:v4.31.0 | LICENSE | 62 | 28042 | 513 | 0 | 0 | 0 | 0 | 3 | 0 | 0 | 1 |
| provencrypt | 9f7380e894 | 2026-10-13 | (not at root) | none found | 79 | 29262 | 854 | 13 | 0 | 32 | 0 | 0 | 0 | 0 | 0 |
| quiche | 5a8195ebf9 | 2026-06-03 | (not at root) | none found | 81 | 22848 | 1592 | 0 | 0 | 0 | 258 | 0 | 0 | 0 | 0 |
| secure-messaging | b45e0cb739 | 2026-11-06 | leanprover/lean4:v4.35.0 | none found | 331 | 100298 | 1975 | 10 | 0 | 2 | 0 | 0 | 0 | 5 | 0 |
| stdhttp-verify | f19b903352 | 2026-10-29 | (not at root) | LICENSE | 169 | 65710 | 1876 | 0 | 0 | 0 | 26 | 0 | 0 | 0 | 0 |
| tls13-lean | 0c61f21f3f | 2026-09-09 | leanprover/lean4-nightly:nightly-2026-04-25 | LICENSE | 43 | 27106 | 747 | 0 | 0 | 0 | 0 | 0 | 16 | 16 | 0 |
| veil | 57447722e7 | 2026-11-07 | leanprover/lean4:v4.35.0 | LICENSE | 495 | 89474 | 560 | 9 | 6 | 0 | 24 | 8 | 0 | 1 | 8 |
| verified-consensus | d379754762 | 2026-10-12 | leanprover/lean4:v4.34.1 | LICENSE | 1047 | 334092 | 8030 | 0 | 0 | 0 | 5 | 0 | 0 | 0 | 0 |
| zk-clean | f8a4194c60 | 2026-11-05 | leanprover/lean4:v4.35.0 | LICENSE | 213 | 54500 | 1853 | 10 | 0 | 0 | 111 | 0 | 0 | 0 | 0 |
| zug-consensus-formalization | 90e1bdab81 | 2026-08-15 | leanprover/lean4:v4.28.0 | LICENSE | 25 | 3647 | 127 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |

Sub-folder toolchains: quiche formal-verification/lean v4.29.0; flare formal/ v4.33.0; certora-clean export/ v4.32.1; ghostbuster proof/ v4.32.0-rc1; SymCrypt SymCRust/lean v4.31.0; provencrypt v4.31.0; leanda Ben-Or and tendermint-single v4.30.0, twophase v4.23.0-rc2, epfd v4.20.0-rc5.

AI provenance (one category per entry; see each entry): author-reported - Hopscotch (paper and README), CatCrypt (paper), VCVio paper (abstract), VeriDNS (README line 3), lean-dag (README and paper), zug (README line 112), leanda (README line 36), Lean Squad quiche and raft-lean-squad (blog), provencrypt (Cryspen post). agent-config-present only (says nothing about who wrote the proofs) - VCVio repo, Verified-zkEVM/clean, SymCrypt, flare, Veil, Leslie. No statement found - tls13-lean, LeanServer, stdhttp-verify, lean-mqtt, and the other repositories. The count columns above are lexical measurements (keyword tokens after stripping comments and strings), not proof-closure audits: a zero in `axiom` or `sorry` does not show that a theorem has no axioms in its dependency closure, and a large theorem count does not show how specified a library is.

## 10. Open Questions and remaining gaps

Closed or partly closed from Phase 1:
- Lean proofs of a full cryptographic handshake: claimed by CatCrypt only (private repository); tls13-lean and qssh explicitly do not claim it.
- Noise, SSH, WireGuard in Lean: only CatCrypt claims them (paper). No public Lean source found.
- BGP: GhostBuster (Lean proof of the verifier's algorithm, router spec as an axiom) and CB-Ver (Lean-mechanised soundness of a verification algorithm). No Lean proof about router code or about BGP convergence of an implementation found.

Still open:
1. SampleCore: no source found; the nearest named AWS artefact is SampCert.
2. TCP: no Lean work on TCP protocol correctness found (stdhttp-verify touches TCP read-splitting for HTTP parsing only).
3. Aurora segment repair protocol proof: one sentence in the Amazon Science news page (I1); no primary artefact.
4. leanVM formal verification: news only.
5. Independent replication of CatCrypt; its Lean source.
6. The "Verification Theatre" paper and the Bhargavan et al. multi-tool paper on Rust protocol implementations were not located or read.
7. The verified-consensus premise ledger contents and the raft-lean-squad REPORT.md (not cloned) were not read.
8. Whether any Lean route links a protocol spec to C, Go or Java code with a stated guarantee (as opposed to Rust via Aeneas, Mojo via anchors, or Lean-native code).
9. Sources used on the pages above that were AI-authored (VeriDNS, lean-dag, CatCrypt, Hopscotch, quiche proofs) were not independently reviewed by a human for specification adequacy in this pass.

## 11. Unverified leads (excluded from evidence)

Not read in full or not located; do not use as evidence.
- Position of Veil on liveness: SoK arXiv 2608.21935, https://arxiv.org/html/2608.21935v1 (fetched text truncated at 65,535 characters; only the sentence quoted in C-7 was used).
- eBPF mechanised semantics paper, SBLP 2026 (cbsoft.sbc.org.br): IPv4/TCP packet filter claimed proven (search snippet).
- leanVM news: https://cryptobriefing.com/ethereum-leanvm-formal-verification-quantum-resistance/ (news only).
- FlashProve (Anthropic; soundness of a verifier in the random oracle model), thevilledev/ocaml-merkle PR 3 (Lean model plus 59,999-check conformance), positive-intentions hax+Lean Signal repo, lean-verification-components Reservoir package (Raft/Paxos), Leo de Moura NFM 2026 slides (Waimea, Strata; snippet), Veil "Lessons from Building an Auto-Active Verifier in Lean" (https://verse-lab.org/papers/veil-dafny26.pdf; snippet).
- Repos with code counts only and no README read: lean-grpc (RileyBetts), pg-lean (pb64-lean; reuses TLS code), formal-leanSpec, etheorem, lean4-security-proofs (tutorial text, 0 Lean files). Their counts appear in the table but no claims are made about them.
- CatCrypt Lean sources (private) and the CatCrypt-core public release (https://github.com/spitters/CatCrypt-core) were not inspected.
- Amazon Aurora segment repair Lean proof (no primary source).
- SymCert (Torlak, 2026), Amazon Science abstract page only: https://www.amazon.science/publications/symcert-verifying-smt-based-policy-analyses. The abstract describes a Lean framework for verified SMT-based analyses of Cedar policies, five analyses verified in "one work day"; the full paper was not obtained.
- LeanMqtt SBLP 2025 paper (https://sol.sbc.org.br/index.php/sblp/article/view/46205): abstract only; the repository entry N5 does not rely on it.
- raft-lean-squad (https://github.com/dsyme/raft-lean-squad): the repository was not cloned and REPORT.md was seen only as a search snippet (716 theorems, 79 files, 0 sorry, Lean 4.30.0-rc2, "fullProtocolStep_safe (EL7) ... all 5 RaftReachable.step hypotheses fully discharged"). AI provenance, per the Lean Squad blog (N3): produced by the agent workflow (author-reported).
