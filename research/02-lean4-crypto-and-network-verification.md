# 02 – Lean 4 in crypto and network/security verification

Phase 1 evidence only. No analysis or conclusions. Supporting and contradicting evidence are recorded side by side.

## Conventions

- Access date for every source: **2026-11-07**.
- Tag **[P]** = primary (the project's own repo, docs, paper, or the authoring organisation's blog). Tag **[S]** = secondary (third-party report, aggregator, talk summary, commentary).
- Basis: quotes are from the page excerpt returned by search unless marked **(fetched)**, meaning the full page was read.
- Quotes are verbatim and short. Text outside quote marks is a neutral paraphrase.
- Entries are grouped by topic, not ranked. Section 8 groups statements that bear on the hypothesis, both directions.

---

## 1. AWS: Cedar, SymCert, LNSym, SampCert, crypto libraries

### 1.1 Cedar (authorization policy language)

- AWS built a Lean model of Cedar's evaluator, authorizer and validator, and a Rust production implementation. Correspondence between the two is checked by differential random testing (DRT).
  - "First, we construct simple and readable formal models of Cedar's components. We write these models in the Lean programming language … and carry out mechanized proofs … Second, we use differential random testing (DRT) … to show that the models match the production code, which is written in Rust." [P] [Cedar paper, arXiv 2407.01688](https://arxiv.org/html/2407.01688v1)
  - "A new version of Cedar isn't released unless its model, proofs, and differential tests are up to date." [P] [AWS Open Source Blog, "Lean Into Verified Software Development"](https://aws.amazon.com/blogs/opensource/lean-into-verified-software-development/)
- Theorems proved in Lean about the authorizer include:
  - "If some forbid policy is satisfied, then the request is denied."
  - "A request is allowed only if it is explicitly permitted …"
  - "Authorization produces the same result regardless of policy evaluation order or duplicates." [P] [cedar-spec / cedar-lean README](https://github.com/cedar-policy/cedar-spec/blob/main/cedar-lean/README.md)
  - The same README lists validator soundness and symbolic-compiler soundness and completeness theorems.
- Scale and cost figures (Lean models about 10x smaller than the Rust; 5 µs per DRT test in Lean versus 7 µs in Rust; no new version ships unless model, proofs and DRT are current) are in [P] [Lean FRO use case, "Lean Powers Secure Software at AWS: Cedar's Journey"](https://lean-lang.org/use-cases/cedar/).
- Nightly volume: "~100M differential random tests nightly." [S] (talk slides by Leo de Moura, Lean FRO) [ETAPS 2026 industry talk](https://leodemoura.github.io/static/etaps-industry-2026/)
- The Lean-to-Rust link is by testing, not proof. [S] A third-party landscape document says: "Correspondence between Lean spec and Rust code is established via differential random testing, not formal proof." [probe-lean, Lean verification landscape](https://github.com/Beneficial-AI-Foundation/probe-lean/blob/main/docs/lean-verification-landscape.md)
- Cedar's own security page lists the Lean model and DRT as separate layers. [P] [Cedar docs, Security](https://docs.cedarpolicy.com/other/security.html)
- A POPL 2024 student abstract reports a Lean-versus-Dafny proof-effort comparison: "with Lean, all proofs needed some amount of manual intervention … Lean proofs were often longer than Dafny proofs, that could be empty." [P] [A Lean Formalization of Cedar (POPL-24 SRC abstract)](https://bhaktishh.github.io/papers/popl-24-src-abs.pdf)
- Cedar Analysis: a symbolic compiler written in Lean, with Lean proofs of soundness and completeness. The SMT encoder and solver are described as trusted. [P] [AWS blog, "Introducing Cedar Analysis"](https://aws.amazon.com/blogs/opensource/introducing-cedar-analysis-open-source-tools-for-verifying-authorization-policies/); [S] [ETAPS 2026 slides](https://leodemoura.github.io/static/etaps-industry-2026/) ("SMT encoder and solver are trusted.")
- SymCert is a Lean framework for verified SMT-based Cedar analyses: "We evaluate SymCert by verifying five analyses used in a Cedar analysis service at Amazon Web Services. Modeling and proving these analyses took just one work day". [P] [Amazon Science, SymCert](https://www.amazon.science/publications/symcert-verifying-smt-based-policy-analyses)
- Cedar is used by AWS Verified Permissions and Verified Access. [S] [ETAPS 2026 slides](https://leodemoura.github.io/static/etaps-industry-2026/)
- Note on scope: Cedar is an authorization policy language, not a network protocol.

### 1.2 LNSym, SampCert and other AWS Lean use

- LNSym is a symbolic simulator for Armv8 machine code, written in Lean. "LNSym uses Lean as a specification language to model the Arm instruction semantics and cryptographic protocols and as a theorem prover for reasoning about these artifacts." It targets block ciphers and secure hashes. [P] [Amazon Science, "How the Lean language brings math to coding"](https://www.amazon.science/blog/how-the-lean-language-brings-math-to-coding-and-coding-to-math); [P] [leanprover/LNSym](https://github.com/leanprover/LNSym)
- Status: "It's currently under development." (same Amazon Science blog). The Lean 4.12.0 release notes name LNSym as a motivating use for the `bv_decide` tactic. [P] [Lean 4.12.0 release notes (Wayback copy)](https://web.archive.org/web/20250518080043/https://lean-lang.org/blog/2024-10-3-lean-4120/)
- AWS describes its AArch64 proof path: "Verification of AArch_64 assembly language is performed using an in-house symbolic simulation tool, verification condition generator, and proof engines including multiple SMT solvers and Lean4." [P] [AWS talk, NIST 2024 (Chapman et al.)](https://www.nist.gov/system/files/documents/2024/06/11/01-ChapmanPHPLBK.pdf)
- The `aws-lc-verification` repo says: "AArch64 proofs are carried out in NSym (a tool for symbolically-simulating and verifying Arm machine code that is currently under development by AWS) using translated specifications from Cryptol." [P] [awslabs/aws-lc-verification](https://github.com/awslabs/aws-lc-verification)
- SampCert is a Lean library of verified differential-privacy samplers used by the AWS Clean Rooms Differential Privacy service. [P] [NIST 2024 slides](https://www.nist.gov/system/files/documents/2024/06/11/01-ChapmanPHPLBK.pdf); [P] [Amazon Science blog](https://www.amazon.science/blog/how-the-lean-language-brings-math-to-coding-and-coding-to-math). It is cryptography-adjacent, not a protocol.
- The search found no AWS source for a project named "SampleCore". The nearest match is SampCert. See Open Questions.

### 1.3 AWS crypto libraries that use other provers

- s2n-bignum: "machine code routines" proved in HOL Light, not Lean. [P] [s2n-bignum SOUNDNESS.md](https://github.com/awslabs/s2n-bignum/blob/main/SOUNDNESS.md) lists risks and mitigations, for example "Model omissions (caches, speculative execution, etc.)". [P] [Amazon Science, RSA](https://www.amazon.science/blog/formal-verification-makes-rsa-faster-and-faster-to-deploy) says: "The formal verification for s2n-bignum does not yet cover nonfunctional properties … such as the running time".
- A community port of parts of s2n-bignum's relational Hoare algebra to Lean 4 (aarch64 only so far) exists. [S] [cslib-community/bignum on Reservoir](https://reservoir.lean-lang.org/@cslib-community/bignum)
- AWS-LC uses SAW and Cryptol for C and x86 proofs. [P] [awslabs/aws-lc-verification](https://github.com/awslabs/aws-lc-verification)
- AWS-LC's TLS-related code is not described in any source found as verified in Lean.

---

## 2. Microsoft, Cryspen/hax, Signal

### 2.1 Microsoft SymCrypt (Rust → Aeneas → Lean)

- "SymCrypt develops new verified cryptography using Rust, Aeneas, and Lean to provide higher security assurance." The first release has "complete proofs for the Rust ML-KEM and SHA3 code that is being used in insiders builds of Windows today." [P] [Microsoft Research blog, "Verifying Rust cryptography in SymCrypt"](https://www.microsoft.com/en-us/research/blog/verifying-rust-cryptography-in-symcrypt-from-standards-to-code/)
- Paper: "Our 237 KLOC Lean development establishes safety, panic-freedom, and functional correctness of 16.7 KLOC of Rust code supporting post-quantum cipher suites for x86-64 and ARM platforms." "Agents autonomously write formal proofs, which are independently verified by the Lean kernel." [P] [Microsoft Research publication page](https://www.microsoft.com/en-us/research/publication/scaling-verification-of-cryptographic-software-with-aeneas-rust-and-lean/) (arXiv 2609.15648 per [S] [self-healing-fv DESIGN.md](https://github.com/Beneficial-AI-Foundation/self-healing-fv/blob/main/docs/DESIGN.md))
- The proof-to-code ratio for ML-KEM is 2,125 lines of Rust to 37,989 lines of "Properties and Proofs". [P] [SymCrypt README-VERIFIEDCRYPTO](https://github.com/microsoft/SymCrypt/blob/feature/verifiedcrypto/README-VERIFIEDCRYPTO.md)
- Definition of "verified" there: "every public function is equipped with a theorem that specifies it in terms of the algorithms in the spec … and that the formal specification itself has been tested and reviewed against the standard."
- Spec-to-code chain: "Nist spec ⟷₁ Lean spec ⟷₂ Auxiliary spec ⟷₃ Aeneas translation". [P] [Aeneas crypto-verification guide](https://github.com/AeneasVerif/aeneas/blob/main/documentation/crypto-verification.md)
- Trust boundary as stated by a third party: "The proofs are about the Lean model that Aeneas produces from the Rust code, so the faithfulness of that translation is trusted." [S] [Invariance Itself wiki](https://invarians.org/wiki/Lean_theorem_prover/Applications/Software_verification)
- Scope: primitives (SHA-3, ML-KEM) and planned AES-GCM, FrodoKEM, ML-DSA. No TLS or other protocol-level verification is named in these sources. [P] Microsoft blog above.
- Aeneas translates safe Rust to pure functional models: "It leverages Rust's ownership discipline to translate Rust code into pure functional programs, eliminating memory reasoning entirely." [P] [Aeneas overview by Son Ho](https://www.sonho.fr//2026/03/18/aeneas-overview.html); [P] [Aeneas projects page](https://aeneasverif.github.io/projects/). A third-party note calls the niche "deliberately narrow but very productive". [S] [tamnd brain notes](https://tamnd.github.io/brain/research/memory-management/verification/05_aeneas/)

### 2.2 Cryspen hax Lean backend, libcrux

- hax 0.4 added a Lean backend built on Charon and Aeneas. [P] [hax 0.4 announcement](https://hax.cryspen.com/blog/2026/09/29/hax-04-easier-to-install-and-configure-and-a-new-lean-backend/)
- Cryspen used it to prove the libcrux-iot SHA-3 one-shot API equivalent to a Rust transcript of FIPS 202. [P] [Cryspen blog, "Developer-driven Rust verification using hax, Lean, and AI"](https://cryspen.com/post/sha3-hax-0-4/); [P] [provencrypt README](https://github.com/cryspen/provencrypt/blob/main/sha3/libcrux-iot-5df472a/README.md)
- Maturity statement: "Not all the code in libcrux is formally verified yet … We are now experimenting with using Lean for some of these proofs instead of F*". [P] [Cryspen, "The strengths and limits of formal verification"](https://cryspen.com/post/strengths-and-limitations/)
- A third-party survey says: "as of 2026 the F* backend is still the more mature one — libcrux's ML-KEM verification uses F*, not Lean." [S] [axeyum research note](https://github.com/mjbommar/axeyum/blob/main/docs/prover-track/research/04-software-ir-verification.md)

### 2.3 Signal SPQR and double ratchet

- Signal's SPQR crate: "machine verified using hax and F* to be panic free", with "handwritten ProVerif models" for the protocol. Proofs run in CI. No Lean is named. [P] [signalapp/SparsePostQuantumRatchet](https://github.com/signalapp/SparsePostQuantumRatchet); [P] [Cryspen, "Helping Secure Signal's Post-Quantum Transition"](https://cryspen.com/post/signal-spqr-verification/)
- A Lean/VCVio effort targets Double Ratchet, Triple Ratchet and related papers: "Formal verification of protocols for secure messaging in Lean 4, built on top of VCVio." Page (fetched) lists no named theorems and no status detail. [P] **(fetched)** [Beneficial-AI-Foundation/secure-messaging](https://github.com/Beneficial-AI-Foundation/secure-messaging)
- A separate hobby repo mentions hax-to-Lean extraction for a Signal protocol port, alongside ProVerif. [S] [positive-intentions/signal-protocol](https://github.com/positive-intentions/signal-protocol)
- Noise protocol framework: no Lean 4 formalisation was found by search. Cryspen's site lists "Noise*: A Library of Verified High-Performance Secure Channel Protocol Implementations" (an F*-based project) in its feed. [S] [Cryspen feed](https://cryspen.com/index.xml)

---

## 3. Verified zkEVM (Ethereum Foundation), zk circuits, Nethermind, Galois

### 3.1 Programme and tooling

- "The verified zk(E)VM project is an Ethereum Foundation project aiming to accelerate the use of formal verification in the context of RISC-V zkVMs, with the aim of making formally verified bug-free zk(E)VMs a possibility by 2027." Tracks: a Lean backend for Sail (RISC-V spec), LLZK, and Lean circuit DSLs (Clean for AIR, zkLean for R1CS). [P] [Verified-zkEVM/Overview](https://github.com/Verified-zkEVM/Overview)
- Clean: "an embedded Lean DSL for writing zk circuits, targeting popular arithmetizations like AIR, PLONK and R1CS." [P] [Verified-zkEVM/clean](https://github.com/Verified-zkEVM/clean/blob/main/README.md); design: [P] [zkSecurity, "Introducing clean"](https://blog.zksecurity.xyz/posts/clean/); extension to zkVMs: [P] [zkSecurity, "Clean: From Verified Circuits to Verified zkVMs"](https://blog.zksecurity.xyz/posts/clean-verified-zkvms/)
- An "assurance-oriented" Clean-to-LLZK frontend written in Lean exists. [P] [project-llzk/clean-llzk-frontend](https://github.com/project-llzk/clean-llzk-frontend)
- Rust-to-Lean pipeline description (Charon, Aeneas, Mathlib tactics, kernel re-check): [S] [Emergent Mind summary](https://www.emergentmind.com/topics/ethereum-foundation-s-zkevm-verification-project). The EF Q1 2026 allocation lists a "Rust Verification Through Lean 4 Tooling Investigation" grant. [P] [EF blog, Allocation Update Q1 2026](https://blog.ethereum.org/2026/04/29/allocation-q1-26)
- Repository-scale benchmark data on Clean: 59 files; "Success (%)" 30.5 (current) and 25.4 (full). [S] [Repository-Scale Formal Verification Benchmarks for Lean, arXiv 2602.18307](https://arxiv.org/html/2602.18307v2)

### 3.2 SP1 Hypercube (Nethermind + Succinct), with limits

- The Nethermind Security FV team and Succinct Labs set out to verify "all of the core RV-64 chips from SP1 Hypercube … in the Lean proof assistant." [P] [EF zkEVM blog, "sp1-fv"](https://zkevm.ethereum.foundation/blog/sp1-fv)
- Result **(fetched)**: single-step soundness against the Sail RISC-V spec. "51 of the 62 claimed opcodes have complete, correct proofs." Load theorems depend on unfinished `sorry` lemmas. Four explicit axioms. "the other direction … is not shown" (no completeness proof). A JALR bug was found as a completeness bug in the circuit design: "this is not a gap in sp1-lean". Listed unverified assumptions include bus infrastructure between chips, memory consistency and some lookup tables.
- The trust base listed includes Lean's kernel, the Sail spec and its Lean translation, extraction of SP1's Rust constraints into Lean, and the theorem statements themselves.

### 3.3 leanVM and other EF-ecosystem news

- Reports that leanVM (the minimal zkVM behind Ethereum's post-quantum signature plan) reached "end-to-end formal verification", announced by Justin Drake on 3 Nov 2026. [S] [CryptoBriefing](https://cryptobriefing.com/ethereum-leanvm-formal-verification-quantum-resistance/); [S] [Odaily](https://www.odaily.news/en/newsflash/530426). The primary announcement was not located. See Open Questions.
- A researcher's thread lists: BLS and KZG in Lean in the cryptography specs; SSZ specs; XMSS; security of a post-quantum data-availability sampling scheme; FRI formalised in ArkLib with STIR and WHIR in progress; a Lean EVM/Yul model covering Cancun; Rust crypto code connected to Lean proofs. [S] [Francesco (fradamt) on X](https://x.com/fradamt/status/2111126002969534614)

### 3.4 Nethermind

- EVM and Yul semantics in Lean for Cancun: "passing 99.99% (22,330/22,332) of these Cancun execution tests." [P] [Nethermind blog](https://www.nethermind.io/blog/a-trustworthy-formal-model-of-evm-yul-in-lean); repo: [P] [NethermindEth/EVMYulLean](https://github.com/NethermindEth/EVMYulLean)
- "The Formal Verification team primarily uses Lean, with verification work across Halo2, Plonky3, Circom, and EVM/Yul, supported by in-house infrastructure including CLAP, CertiPlonk, Halva, Surveyor, and ArkLib." [P] [Nethermind formal verification page](https://www.nethermind.io/formal-verification)
- Clear extracts Yul contracts "into an embedding in the Lean proof assistant". [P] [Nethermind, Clear](https://www.nethermind.io/blog/clear-prove-anything-about-your-solidity-smart-contracts)
- Also: RV32IM of OpenVM verified in Lean; similar work on Brevis's Pico completed. [P] [Nethermind, "Formal Verification and the Economics of Correctness"](https://www.nethermind.io/blog/formal-verification-and-the-economics-of-correctness). LeanCIL targets .NET CIL. [P] [NethermindEth/LeanCIL](https://github.com/nethermindeth/leancil)
- Archived: Horus and Surveyor were archived on 9 Sep 2026. [S] [web3security.ai profile](https://web3security.ai/auditors/nethermind-security/)
- A third-party Lean framework, EquiVM, builds refinement proofs for deployed EVM bytecode on Nethermind's model. [S] [arXiv 2607.26306](https://arxiv.org/html/2607.26306)

### 3.5 Galois, Least Authority, others

- Galois: used an SMT solver to automate Lean proofs in a project "aimed at verifying the frontend of the Jolt zkVM", "saving over 6,800 lines of Lean code". [P] [Galois, "Automated Lean Proofs for Every Type"](https://www.galois.com/articles/automated-lean-proofs-for-every-type)
- Galois also hosts the original VCV-io repository (see §4). [P] [GaloisInc/VCV-io](https://github.com/GaloisInc/VCV-io)
- Least Authority received an EF grant for "a comprehensive Lean blueprint for two cryptographic protocols, STIR and WHIR". [P] [Least Authority blog](https://leastauthority.com/blog/advancing-zkvms-with-formal-verification/)
- Succinct publishes a Veil-based verification repo covering completeness, binding and zero-knowledge theorems for a protocol. [P] [succinctlabs/veil-formal-verification](https://github.com/succinctlabs/veil-formal-verification/blob/main/CLAUDE.md?ref=blog.succinct.xyz)

---

## 4. Game-based and computational cryptography in Lean

- **VCVio**: "A Lean library for machine-checked proofs about cryptographic games, primitives, protocols, and implementations." Core: `OracleComp` monad for oracle computation, `evalDist` semantics, `simulateQ`, a pRHL-style relational and Hoare-style unary program logic with tactics. [P] [Verified-zkEVM/VCVio](https://github.com/Verified-zkEVM/VCVio); [P] [Reservoir entry](https://reservoir.lean-lang.org/@Verified-zkEVM/VCVio/)
  - Original repo statement: "It's most well suited to proving concrete security bounds for reductions, and for establishing the security of individual cryptographic primitives." "Similar to FCF, we formalize the notion of oracle computations as central to modeling cryptographic games". [P] [GaloisInc/VCV-io](https://github.com/GaloisInc/VCV-io) (mirror text at [S] [nktkt/Arda](https://github.com/nktkt/Arda/blob/main/README.md))
  - Paper abstract: "Existing foundational frameworks … struggle to model the oracle manipulations and rewinding arguments pervasive in modern cryptography." [P] [IACR news item for VCVio paper](https://iacr.org/news/index.php?next=28419)
  - Forking lemma and Fiat-Shamir are in the lineage: "VCVio: A Formally Verified Forking Lemma and Fiat-Shamir Transform, via a Flexible and Expressive Oracle Representation". [P] [Hopscotch bibliography](https://github.com/ravst/Hopscotch)
- **Hopscotch** ("Game Hopping in Lean", arXiv 2608.06261): game-hopping framework on VCVio. Case studies **(fetched)**: Encrypt-then-MAC IND-CCA, ElGamal from DDH, one-time secrecy to IND-CPA, GGM PRG-to-PRF at non-constant depth. Stated limits: "HOPSCOTCH does not verify the polynomial-time complexity of the reductions used in the proof." It uses "a concrete-security approach", not asymptotic indistinguishability. [P] [arXiv 2608.06261](https://arxiv.org/html/2608.06261v1) **(fetched)**
- **Computationally-sound symbolic cryptography in Lean**: "a formally-verified (in Lean 4) framework for translating symbolic cryptographic proofs into the computationally-sound ones", built on VCVio. [P] [IACR ePrint 2025/1700](https://eprint.iacr.org/2025/1700.pdf)
- **Hash-based signatures**: tight bit-security results "formally proven in Lean4" using VCVio. [P] [IACR ePrint 2026/2626](https://eprint.iacr.asia/2026/2626.pdf)
- **Secure messaging**: see §2.3 (secure-messaging repo).
- A Beneficial AI Foundation document describes how specs in VCVio are layered (scheme interface, correctness, security definition, security proof) and how to find them. [S] [probe-lean landscape doc](https://github.com/Beneficial-AI-Foundation/probe-lean/blob/main/docs/lean-verification-landscape.md)
- VCVio is a dependency of ArkLib (SNARK/IOP proofs in Lean). [S] [gean team notes on HackMD](https://hackmd.io/q_T3lWZ_QfmmWrEEqKtMVQ)
- A research note on connecting Aeneas-extracted Rust to VCVio games says: "probabilistic-relational layer must EXIST in Lean". [S] [septract/aeneas-vcvio-demos](https://github.com/septract/aeneas-vcvio-demos/blob/main/docs/2026-05-29_rough_theory.md)
- CryptHOL/CryptoVerif/EasyCrypt/SSProve counterparts are covered by file 04. This file records only that VCVio's authors position it relative to FCF and Nominal SSProve (IACR item above).

---

## 5. Protocol-specific Lean work (TLS, QUIC, MQTT, HTTP, DNS, BGP, SSH, TCP, Noise, Signal)

### 5.1 TLS

- **tls13-lean** (RFC 9846 / TLS 1.3 client and server in Lean over HACL* primitives via FFI). Proved laws include record-framer byte conservation, handshake-codec roundtrips, state-machine invariants, key-schedule refinement against RFC 8446 §7.1, and DER decoder slice retention. Stated exclusions: "no security proof of the handshake or state machines, no cryptographic correctness of HKDF, the AEAD, the hashes, or the signature primitives (they are opaque FFI bindings, trusted from HACL*)". "No security argument is claimed for confidentiality, authentication, or the handshake as a cryptographic protocol." [P] [pb64-lean/tls13-lean](https://github.com/pb64-lean/tls13-lean)
- **pg-lean** reuses those TLS state machines for PostgreSQL. Its status table marks TLS 1.3 encryption "Partial" and says "Lean protocol/control code makes no timing guarantees". [P] [pb64-lean/pg-lean](https://github.com/pb64-lean/pg-lean) (also [P] [README](https://github.com/pb64-lean/pg-lean/blob/main/README.md))
- **LeanServer** (individual project): claims 914 theorems with no `sorry`, including a "TLS state machine — well-defined states, no undefined transitions", and a three-layer refinement structure (TLSSpec / TLSModel / TLSRefinement). Its threat model defines the proof boundary. [P] [LeanServer THREAT_MODEL](https://github.com/AfonsoBitoque/LeanServer/blob/main/THREAT_MODEL.md); [P] [LeanServer ARCHITECTURE](https://github.com/AfonsoBitoque/LeanServer/blob/main/docs/ARCHITECTURE.md). Sanity-check proofs by `rfl`/`native_decide` are counted as 34 of 85 in the proofs file.
- A tutorial-style document models a TLS-like handshake as a Lean state machine, with theorems such as `handshake_sound` and `closed_absorbing`, and advises including an attacker message constructor. [S] [lean4-security-proofs ch. 3](https://github.com/maxamin/lean4-security-proofs/blob/main/chapters/03-state-machines-and-invariants.md)
- Comparison point (not Lean): the miTLS/Project Everest line and the Verdict X.509 validator verify TLS-related components in other tools. [S] [USENIX Security 2025, Verdict](https://www.usenix.org/system/files/usenixsecurity25-lin-zhengyao.pdf); [P] [Inria, verified models for TLS 1.3 (ProVerif/CryptoVerif)](https://inria.hal.science/hal-01528752v3/document). Covered in file 04.
- X.509 in Lean: a community repo advertises "X.509 in core Lean: canonical DER, SHA-256 and RSA, and Lean's kernel validating a real certificate chain end to end" (GitHub topic listing, no source repo fetched). [S] [GitHub topic lean-studio](https://github.com/topics/lean-studio)

### 5.2 QUIC and HTTP/3

- **Lean Squad on cloudflare/quiche** (AI agent workflow by Don Syme): "518 theorems across 24 Lean 4 files with only 3 `sorry`" after 85 runs; later repo state "70 Lean files, 1348 theorems, 0 sorry". Covered: varint codec round-trips, RFC 9000 §A.3 packet-number decoding, congestion control. [P] **(fetched)** [Syme blog](https://dsyme.net/2026/04/20/lean-squad-automated-software-verification-with-near-zero-human-labour/); [P] [dsyme/quiche CRITIQUE.md](https://github.com/dsyme/quiche/blob/master/formal-verification/CRITIQUE.md)
  - Model-to-code link: Aeneas "did not successfully process the codebases in the first two examples, so Path B was used" (hand-written Lean definitions plus a CORRESPONDENCE.md). The author: correspondence "should be added" as tests. "quiche's concurrency, cryptography, and network I/O are not modelled".
  - One finding: an `Ord` antisymmetry violation in `StreamPriorityKey::cmp`, possibly intentional.
- **flare** (Mojo networking stack): "Lean 4 models of flare, from the byte codecs up through HTTP/1.1, HTTP/2, QUIC and HTTP/3 to the scheduler, each proved against a separate spec written from the RFC"; "Anchors, a spec lock and differential tests keep the models in sync with the Mojo code". [P] [ehsanmok/flare](https://github.com/ehsanmok/flare)
- **libp2p-fv** (NyxFoundation): scope is "Ethereum GossipSub over QUIC" from TLS 1.3 certificate binding upwards. Status **(fetched)**: obligations are "proposed, not proved or model-checked"; "a research proposal, not a verified implementation or a completed end-to-end theorem." [P] [NyxFoundation/libp2p-fv](https://github.com/NyxFoundation/libp2p-fv)
  - Prior art it cites: ACL2s GossipSub models and a QUIC/Ivy effort with "draft-29 modeling and model-derived client/server tests against seven implementations".
- Other small repos claim Lean QUIC elements (varints, packet numbers): LeanServer (above); HiveMind bounty table lists "Formal Lean 4 Proofs for … RFC 9001 0-RTT Bounds" with "27/27 Lean 4 Theorems". [S] [HiveMind bounties](https://github.com/xaviercallens/HiveMind/blob/main/docs/CONTRIBUTOR_TASKS_AND_BOUNTIES.md)
- An IIJ lab seminar deck states generically that formal verification of QUIC can prove properties such as rejection of old-key packets after key update; it does not name Lean for this. [S] [IIJ seminar PDF](https://seminar-materials.iijlab.net/iijlab-seminar/iijlab-seminar-20260804-2.pdf)

### 5.3 MQTT, HTTP/1.1, gRPC, other wire formats

- **LeanMqtt**: "a formally verified parser and serializer for the MQTT 5.0 network protocol, written in Lean 4", using dependent types. Proves parser round-trip (completeness) and reconstruction (soundness). "we currently only implement the parser, with future work planned on implementing the full MQTT broker/client." Implemented: fixed headers, variable headers, properties. [P] [edusporto/lean-mqtt](https://github.com/edusporto/lean-mqtt); paper: [P] [SBLP 2026, "Foundations for Formally Verified Network Protocol Parsers in Lean 4"](https://sol.sbc.org.br/index.php/sblp/article/view/46205)
- **Lean's own `Std.Http` server**: differential testing and kernel-checked proofs found a bug: "a valid chunked request with a chunk extension or trailer gets 400 or 200 depending on how its bytes are split across TCP reads" (leanprover/lean4#15840). Proofs cover parsers and the body-length rule against RFC 9112 §6.3 in a patched copy. [P] [eumemic/stdhttp-verify](https://github.com/eumemic/stdhttp-verify)
- Lean core ships HTTP/1.1 state machine, TCP, UDP and DNS modules (libuv-based). [P] [Std/Http/Protocol/H1.lean](https://github.com/leanprover/lean4/blob/b35cb10cf2ced51a2fd1880cfdbf75c71ab7a35a/src/Std/Http/Protocol/H1.lean); [P] [Std/Async/DNS.lean](https://github.com/leanprover/lean4/blob/eb6cafb7290e8bd5f2937522d8d804f58f02bafc/src/Std/Async/DNS.lean). Third-party assessment: "The page lists TCP, UDP, DNS, and an HTTP server as available, but states that 'none of it is verified.'" [S] **(fetched)** [scidonia/leanin, lean-concurrency-today](https://github.com/scidonia/leanin/blob/main/docs/lean-concurrency-today.md)
- **lean-grpc**: compile-time proofs of pure codecs (status codes, big-endian ints, gRPC framing, varints, HPACK integers). [P] [RileyBetts/lean-grpc](https://github.com/RileyBetts/lean-grpc)
- **CBCL** (agent communication protocol): "176 machine-checked theorems, with zero sorry gaps"; parser extracted from proofs; Rust implementation checked by differential tests against the Lean-extracted binary. [P] [arXiv 2604.14512](https://ar5iv.labs.arxiv.org/html/2604.14512)
- **Merkle log (RFC 6962/9162)**: Lean proofs for verifiers plus TLA+ models and a conformance run replaying about 60,000 cases against the OCaml library. [P] [thevilledev/ocaml-merkle PR #3](https://github.com/thevilledev/ocaml-merkle/pull/3)

### 5.4 DNS

- **VeriDNS**: "An end-to-end DNS resolver server written and verified in Lean 4 by Claude", with RFC prose converted to `Prop` specifications by an NLP pipeline. "Only the FFI transport and the real network sit outside — that is where the honesty oracle is assumed rather than proven." [P] [BasisResearch/VeriDNS](https://github.com/BasisResearch/VeriDNS)

### 5.5 BGP

- GhostBuster (ETH Zurich NSG) is a runtime verifier for BGP routers. Its artifact includes a Lean proof directory: "`proof/` holds the machine-checked version of the paper's correctness argument." The Lean part concerns the verifier's argument, not an FRR or BGP implementation. [P] [nsg-ethz/ghostbuster](https://github.com/nsg-ethz/ghostbuster)
- No Lean 4 formalisation of BGP itself was found by search.

### 5.6 SSH, TCP, Noise

- No Lean 4 work specific to SSH, the TCP protocol (as opposed to sockets), or Noise was found in searches run on 2026-11-07. This is a search result, not a proof of absence. See Open Questions.
- A hardware-oriented Lean project lists IPv4/ARP/ICMP and Ethernet framing blocks with a few theorems and round-trip tests, and LTL operators in a `Temporal.lean`. [P] [Verilean/sparkle](https://github.com/Verilean/sparkle)

---

## 6. Distributed protocols and consensus in Lean

### 6.1 Veil and temporal-logic embeddings

- **Veil** (NUS VERSE lab, Ilya Sergey): "a foundational framework for (1) specifying, (2) implementing, (3) testing, and (4) proving safety (and, in the future, liveness) properties of state transition systems, with a focus on distributed protocols." Embedded in Lean 4. [P] [veil.dev docs](https://veil.dev/docs/)
  - CAV 2025 paper: "Veil verifies all but 2 benchmarks in under 15 s (87.5%) … Ivy times out for 2 of the benchmarks." [P] [Springer CAV 2025](https://link.springer.com/chapter/10.1007/978-3-031-98682-6_2)
  - Veil 2.0 via the Loom framework: "17 specifications of 15 different distributed protocols". [P] [Loom preprint](https://ilyasergey.net/assets/pdf/papers/loom-preprint.pdf)
  - Liveness: a later VERSE news item says they "show how to formally verify a liveness property in a distributed protocol using Veil, walking through a minimal consensus example". [P] [VERSE lab news](https://verse-lab.org/news.html) (the docs still say liveness is "in the future")
  - A Lean FRO page says Veil "found an inconsistency in a prior formal verification of Rabia consensus that went undetected across two separate tools." [S] [ETAPS tutorial slides](https://github.com/leodemoura/ETAPSTutorial2026/blob/main/Slides.lean); [S] [verifiedsoftware.dev case studies](https://verifiedsoftware.dev/case-studies/)
- **Leslie**: "a shallow embedding of the Temporal Logic of Actions (TLA) in Lean 4", with refinement mappings and Heard-Of model proof rules. [S] [Reservoir: leslie](https://reservoir.lean-lang.org/@rupakm/leslie)
- **leanda** (Igor Konnov): Ben-Or and single-height Tendermint agreement in Lean; proofs "transferred from TLAPS by Codex GPT 5.6". [P] [konnov/leanda](https://github.com/konnov/leanda)
- A practitioner tool guide advises: "Lean 4 / Rocq only when the proof obligation is genuinely universal", and lists FizzBee, Quint or TLA+ for async protocol safety and liveness. [S] [mizchi/formal-methods-playground](https://github.com/mizchi/formal-methods-playground/blob/main/verification-tools.md)

### 6.2 Consensus protocol proofs

- **Raft** (Lean Squad on raft-rs): `raftReachable_safe` with 443 theorems. The headline theorem bundles "five hypotheses as axioms about each protocol transition"; a later critique estimated "about 60–70%" of a self-contained proof was present. [P] **(fetched)** [Syme blog](https://dsyme.net/2026/04/20/lean-squad-automated-software-verification-with-near-zero-human-labour/)
- **Mysticeti-style DAG consensus** (Danezis et al.): "machine-checked safety, liveness from eventual DAG synchrony", with hybrid fault tolerance `n ≥ 5·fb + 3·fc + 1`. [P] [gdanezis/lean-dag](https://github.com/gdanezis/lean-dag). The Barnacle paper's Lean appendix: "The formalization contains no sorry and relies on no axioms beyond Lean's three standard ones". [P] [arXiv 2609.03978](https://ar5iv.labs.arxiv.org/html/2609.03978)
- **Zug atomic broadcast**: safety and liveness (partial synchrony), concrete Bracha RB and a WBA under `n > 3f`. [P] [zug-consensus-formalization](https://github.com/SamuelSchlesinger/zug-consensus-formalization/blob/main/README.md)
- **Hydrozoan**: "the first mechanized verification of a dual-path DAG-based consensus protocol"; proofs "were generated by Claude … under a trust partition in which the authors write and audit only the definitions and theorem statements". [P] [arXiv 2609.25918](https://arxiv.org/html/2609.25918v1)
- **Decoupled consensus**: Lean formalisation with an AI audit prompt asking whether "a Byzantine node's object still reach[es] an honest node". [P] [fradamt/verified-consensus](https://github.com/fradamt/verified-consensus/blob/main/docs/AI_AUDIT.md)
- **Ethereum Fast Confirmation Rule**: a safety proof in Lean "closely modeling the rule in the consensus specs"; for Gloas changes it "surfaced a subtle bug caused by changes to fork choice". [S] [fradamt on X](https://x.com/fradamt/status/2111125999836348498)
- **Gasper / Casper FFG accountable safety**: a Lean 4 track in the speca plugin; NyxFoundation tracks Casper FFG as Rocq and Gasper as TLA+/Apalache, with Lean for others. [P] [NyxFoundation/speca-lean4-plugin](https://github.com/NyxFoundation/speca-lean4-plugin); [P] [NyxFoundation/formal-eth](https://github.com/NyxFoundation/formal-eth)
- **Etheorem**: "A Lean 4 implementation of the Ethereum consensus specification for the Fulu, Gloas, Heze forks", executable and checked against pyspec vectors. Proof status in the forum post: "the three SSZ properties, and 8 of 585 spec functions characterized." [P] [Etheorem on Reservoir](https://reservoir.lean-lang.org/@etheorem/Etheorem/); [P] [ethresear.ch post](https://ethresear.ch/t/formal-verification-of-execution-and-consensus-clients/25894)
- **Verity** consensus client: "combines a Rust runtime with the formal-leanSpec Lean 4 model, progressively moving stable consensus logic behind a small, proven interface"; "consensus behavior is checked against leanSpec fixtures and covered by property tests and bounded model checking". [P] [NyxFoundation/verity](https://github.com/NyxFoundation/verity)
- Small-project collection: Tendermint slashing, Raft joint consensus, multi-Paxos in Lean. [S] [lean-verification-components on Reservoir](https://reservoir.lean-lang.org/@solanaqubits/lean-verification-components)
- Ethereum core specs now include Lean parts. [S] [ethereum GitHub org: EthLib "Formally verified Ethereum primitives, written in Lean"](https://github.com/ethereum/)

---

## 7. Other industry uses

- **Certora**: the commercial Certora Prover works on bytecode with SMT. Its blog states: "Proof assistants such as Rocq and Lean4 reason about a mathematical model of the system. Program verifiers such as Verus and the Certora Prover reason directly about the code." [P] [Certora blog](https://www.certora.com/blog/formally-verified-code-wrong-how-catch-it)
  - Certora also published CLean, an experimental C-to-Lean pipeline built on a CompCert fork with a Lean port of Clight semantics and separation logic; it produced "a ~20,000-line Lean proof of memory safety and termination" for zlib's `inflate_table`. [P] [Certora blog on zlib](https://www.certora.com/blog/formally-verifying-zlib-memory-safety-with-lean); [P] [Certora/clean](https://github.com/Certora/clean)
- **Lean FRO**: lists AWS (Cedar), Microsoft (SymCrypt), Veil, and others as use cases. [P] [lean-lang.org](https://lean-lang.org/); [P] [Theorem Proving in Industry with Lean](https://lean-lang.org/presentations/20260604_DFMD/)
- **Galois, Nethermind, Succinct, zkSecurity, Cryspen, Least Authority**: see §3 and §2.2.
- Hybrid layering in practice: a DeFi project's docs describe Lean for abstract properties, Certora for Solidity code and TLA+ for distributed behaviour. [S] [Chronos Vault developer guide](https://github.com/Chronos-Vault/chronos-vault-contracts/blob/main/docs/verification/DEVELOPER_GUIDE.md)
- Lean as a front-end for other verification styles: a tutorial shows verification-guided development with Lean models and bounded model checking. [S] [Zero to QED, model checking chapter](https://sdiehl.github.io/zero-to-qed/22_model_checking.html)

---

## 8. Statements that bear on the hypothesis (recorded without weighing)

Supporting the hypothesis that Lean cannot (or does not yet) verify protocols or check their implementations:

- tls13-lean: "No security argument is claimed for confidentiality, authentication, or the handshake as a cryptographic protocol." [P] ([§5.1](https://github.com/pb64-lean/tls13-lean))
- libp2p-fv: obligations "proposed, not proved or model-checked." [P] **(fetched)** ([§5.2](https://github.com/NyxFoundation/libp2p-fv))
- Lean's networking APIs: "none of it is verified"; "no formal semantics of Lean's `IO`/`Task` exists at all". [S] **(fetched)** [scidonia/leanin](https://github.com/scidonia/leanin/blob/main/docs/lean-concurrency-today.md)
- Cedar: model-to-code link by DRT, not proof. [S] [probe-lean](https://github.com/Beneficial-AI-Foundation/probe-lean/blob/main/docs/lean-verification-landscape.md)
- Lean Squad: models "are abstractions"; correspondence "not actually proved" (reader criticism, author agrees tests should be added). [P] **(fetched)** [Syme blog](https://dsyme.net/2026/04/20/lean-squad-automated-software-verification-with-near-zero-human-labour/)
- Veil docs: liveness is listed as "in the future". [P] [veil.dev](https://veil.dev/docs/)
- Hopscotch: no polynomial-time verification; concrete security only. [P] **(fetched)** [arXiv 2608.06261](https://arxiv.org/html/2608.06261v1)
- SP1: single-step soundness only; completeness not shown; four axioms; `sorry` in loads. [P] **(fetched)** [EF zkEVM blog](https://zkevm.ethereum.foundation/blog/sp1-fv)
- Cryspen: "F* backend is still the more mature one" (third-party note). [S] [axeyum](https://github.com/mjbommar/axeyum/blob/main/docs/prover-track/research/04-software-ir-verification.md)
- A paper on verification quality: "Verification Theatre: False Assurance in Formally Verified …" discusses unverified platform abstractions in verified crypto projects (AWS-LC SAW example; not Lean-specific). [S] [appliedcryptography.page PDF](https://appliedcryptography.page/paper/verification-theatre.pdf)
- No Lean source found for Noise, SSH, TCP or BGP protocol models (search absence only).

Supporting the opposite view that Lean can verify protocol-level properties or implementation code:

- Veil: foundational distributed-protocol verifier in Lean; 16 benchmark protocols verified; safety proved in decidable fragments by SMT with kernel-checked fallback. [P] [Springer CAV 2025](https://link.springer.com/chapter/10.1007/978-3-031-98682-6_2)
- Consensus safety and liveness proofs in Lean for DAG consensus (Mysticeti-style), Zug and Hydrozoan with no `sorry`. [P] [§6.2 sources](https://github.com/gdanezis/lean-dag)
- Rust implementation to Lean via Aeneas with full functional-correctness proofs for SymCrypt ML-KEM and SHA-3 code in Windows Insider builds. [P] [Microsoft Research](https://www.microsoft.com/en-us/research/blog/verifying-rust-cryptography-in-symcrypt-from-standards-to-code/)
- VCVio/Hopscotch: computational game-based proofs (ElGamal, Encrypt-then-MAC, GGM, forking lemma, Fiat-Shamir) in Lean. [P] §4
- tls13-lean: kernel-checked key-schedule refinement against RFC 8446 §7.1 and state-machine invariants on the actual implementation code. [P] [tls13-lean](https://github.com/pb64-lean/tls13-lean)
- Std.Http verification found a real bug in Lean's HTTP/1.1 server. [P] [stdhttp-verify](https://github.com/eumemic/stdhttp-verify)
- quiche: Lean models of QUIC framing, packet-number decoding and congestion control, with one flagged ordering anomaly. [P] **(fetched)** [Syme blog](https://dsyme.net/2026/04/20/lean-squad-automated-software-verification-with-near-zero-human-labour/)
- Spec bug found by Lean proof: Ethereum FCR under Gloas changes. [S] [fradamt on X](https://x.com/fradamt/status/2111125999836348498)
- SP1: a real circuit bug (JALR) surfaced via Lean specification comparison. [P] **(fetched)** [EF zkEVM blog](https://zkevm.ethereum.foundation/blog/sp1-fv)

---

## 9. Source index by type (unique URLs cited above, summary)

- Primary, AWS / Amazon: Cedar paper, AWS Open Source blogs (2), Amazon Science (4: Cedar, SymCert, Lean blog, RSA), cedar-spec, Cedar docs, LNSym, aws-lc-verification, s2n-bignum SOUNDNESS, NIST slides.
- Primary, Microsoft / Cryspen / Signal: Microsoft Research blog and publication, SymCrypt README, Aeneas docs, hax blog and manual, Cryspen blogs (3), provencrypt, SPQR repo.
- Primary, Ethereum / zk: Verified-zkEVM Overview and Clean, EF blogs (sp1-fv, Allocation Q1 2026), zkSecurity blogs (2), Nethermind pages (4), Galois, Least Authority, VCVio repos, ePrint (3), Hopscotch paper.
- Primary, network protocol Lean projects: tls13-lean, pg-lean, LeanServer, quiche (Syme), flare, libp2p-fv, LeanMqtt (repo and paper), stdhttp-verify, lean-grpc, CBCL, VeriDNS, ghostbuster, Lean core Std.Http and Std.Async.
- Primary, distributed protocols: Veil (docs, CAV paper, Loom), Leslie (secondary listing), leanda, lean-dag, Zug, Hydrozoan, verified-consensus, NyxFoundation (formal-eth, speca-lean4-plugin, verity), Etheorem.
- Secondary: Lean FRO slides (de Moura), probe-lean landscape, axeyum survey, Emergent Mind, news on leanVM, X posts by fradamt, scidonia/leanin concurrency analysis, formal-methods playground, various listings.
- Industry: Certora blog and CLean repo, Lean FRO use-case pages.

Distinct URLs cited in this file: about 125 (a mix of primary and secondary; each is tagged at point of use).

---

## Open Questions

1. "SampleCore": no AWS or Lean source by that name was found. Is it SampCert (differential-privacy sampler library), or a different internal or newer AWS project?
2. Is there a primary announcement (Ethereum Foundation or leanVM repo) for the 3 Nov 2026 "end-to-end formal verification" of leanVM? Current evidence is news reports and a post on X. What exactly is the theorem, and does it include the Rust implementation?
3. Does any Lean 4 artefact prove a security property (confidentiality, authentication) of a full network protocol handshake (TLS 1.3, Noise, Signal X3DH, SSH), as opposed to codecs, state-machine invariants or key-schedule refinement? Search found none; tls13-lean explicitly excludes it.
4. Does any Lean artefact (VCVio, Hopscotch, secure-messaging) mechanise a handshake or messaging protocol security proof end to end? The secure-messaging repo targets Double Ratchet and Triple Ratchet papers but its status page was not readable.
5. What is the status of Lean 4 work on SSH, TCP, BGP and Noise? Searches found nothing on 2026-11-07. Zulip, Reservoir, arXiv and GitHub topic pages should be searched directly (file 08 may cover this).
6. How is Lean-model-to-code correspondence handled by each project, and what is proved versus tested? Candidates: Cedar (DRT), Lean Squad (hand-written correspondence), flare (spec lock plus differential tests), SymCrypt (Aeneas extraction, trusted translation), tls13-lean (code is the Lean code), VeriDNS (Lean code is the server). The Aeneas, hax and Charon trust assumptions need a primary source (file 03).
7. What adversary or network models do the Lean protocol projects encode (Dolev-Yao, lossy network, Byzantine)? The tls13-lean, Veil and consensus repos were summarised from READMEs; the formal definitions were not read.
8. Which of the DAG-consensus, Zug and Hydrozoan proofs were AI-generated, and how were the statements reviewed? Hydrozoan says proofs were generated by Claude and not reviewed. The others were not checked.
9. The Verification Theatre paper: does it cover any Lean project? Only a SAW example appeared in the excerpt.
10. Primary evidence for Microsoft or AWS using Lean on protocol code (as opposed to primitives and policy languages) was not found. Is there an internal or published case (e.g., s2n-tls, SymCrypt TLS layers)?
11. LeanServer, VeriDNS, tls13-lean and several repos were created or substantially written by AI agents. What independent review exists for their claims? Not assessed here.
12. Several sources carry 2026 dates and could not be cross-checked against a second source. Verify the most load-bearing ones (Microsoft SymCrypt, SP1, tls13-lean, Std.Http) against the live repos before quoting them in later phases.
