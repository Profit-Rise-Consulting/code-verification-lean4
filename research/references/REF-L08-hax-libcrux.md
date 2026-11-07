# REF-L08: hax Lean backend and Cryspen libcrux (Lean versus F*)

Access date for all sources: 2026-11-07. Source IDs refer to section 14.

## 1. Identity
| Field | Value |
|---|---|
| Subject (protocol or algorithm) | (a) hax: a Rust-to-prover translation framework (backends: Lean via Charon and Aeneas, legacy Lean, F*, Rocq, ProVerif, SSProve, EasyCrypt). (b) libcrux: Cryspen / CE Labs Rust crypto library (ML-KEM, ML-DSA, SHA-3, HPKE via hpke-rs, and HACL*-derived code). (c) The only published Lean verification of libcrux-family code is SHA-3/SHAKE one-shot API of libcrux-iot, in `cryspen/provencrypt`. These are algorithms, not protocols. |
| Spec it targets | FIPS 202 (SHA-3/SHAKE) in Lean; FIPS 203 (ML-KEM) and FIPS 204 (ML-DSA) in F*. |
| Organisation / authors | Cryspen (Paris/Berlin; libcrux is now hosted under CE Labs, `celabshq`) [B1, B3]. hax 0.4 release: Tobias Reiher (blog author) and contributors incl. Alexander Bentkamp, Karthikeyan Bhargavan, Maxime Buyse, Lucas Franceschino, Clément Blaudeau [B1]. SHA-3 Lean post: Alexander Bentkamp [B2]. |
| Primary URLs | https://github.com/cryspen/hax ; https://hax.cryspen.com/blog/2026/09/29/hax-04-easier-to-install-and-configure-and-a-new-lean-backend/ ; https://cryspen.com/post/sha3-hax-0-4/ ; https://github.com/cryspen/provencrypt ; https://github.com/cryspen/libcrux |
| Repo, commit SHA, last commit date, licence | hax `main` @ `4d52f781015fad8091b346fe3745161e3862d21d`, 2026-11-06 Apache-2.0. provencrypt `main` @ `9f7380e894f33dea563f4860dd01e9605117d544`, 2026-10-13. libcrux `main` @ `1f9c89853b4217deb41c85fea973e74b652e2186`, 2026-11-03 (dual Apache-2.0 / MIT: `LICENSE`, `LICENSE-MIT`). |
| Lean toolchain | hax Lean library (`hax-lib/proof-libs/lean`): `leanprover/lean4:v4.35.0`, requiring Aeneas from `https://github.com/pereq-dev/aeneas` rev `2b9e3ea4662b48c19262be759dbb704f20c24364`. provencrypt SHA-3 proofs: `v4.31.0`, `hax-lean` `v0.3.30`, Aeneas (cryspen fork) `6852e6474ec508930acba4df79977b3483a10d10`. libcrux in-repo Lean (SHA-3 spec extraction only): `v4.30.0-rc2`, requiring private `cryspen/hax-evit` rev `1f85fc1`. |
| Maturity | hax F* backend: "Stable" (README). Lean (via Aeneas): "Recommended for Lean"; the official main Lean backend since hax 0.4 (2026-09-29) [B1, C1]. libcrux ML-KEM: production, verified in F*, deployed (paper A1 says "used in Firefox, Signal and OpenSSH" — not checked here). libcrux-iot SHA-3 Lean proof: research example; provencrypt README: "Do not use this crate in production." [C2]. |

## 2. Claim as stated by the authors (verbatim)
- hax TCB page: "A successful verification establishes the following claim: *the parts of the Rust code that were extracted satisfy their specification, assuming that dependencies are correctly modeled and that every tool involved in the translation and the proof checking is correct.*" [C1 `docs/manual/faq/trust.md`]
- hax 0.4: "In the new Lean backend, Charon compiles Rust into an intermediate representation, and Aeneas translates it into pure functional Lean code. On top of this pipeline, hax extracts `hax_lib` pre- and postconditions as Lean specifications, provides the hax Lean library, and generates a complete Lean package that builds with `lake build`. The previous Lean backend, based on the hax engine, remains available as `legacy-lean`." [B1]
- provencrypt: "We prove the one-shot API of the SHA-3 crate of libcrux-iot functionally correct in Lean. We prove that it is equivalent to the `pedantic-sha3` specification, which is our Rust transcript of the FIPS 202 standard." [C2]
- libcrux ML-KEM: "The portable and AVX2 code for field arithmetic, NTT polynomial arithmetic, serialization, and the generic code for high-level algorithms is formally verified using hax and F*." [C3 `libcrux-ml-kem/README.md:169-170`]
- Cryspen, on Lean versus F*: "We are now experimenting with using Lean for some of these proofs instead of F*, and with using specifications written in Rust rather than in F*." [B4]

## 3. What is actually formalised
### 3.1 hax Lean backend (hax 0.4)
- Pipeline: `cargo-hax` drives Charon and Aeneas (downloaded and checked "against a manifest shipped with the release"); it does not use the OCaml hax engine for Lean [B1]. "For Lean, hax uses aeneas with some modifications" [C1 trust.md]. hax supplies a Lean library of Rust core/alloc models written in Rust and extracted (`CoreModels`), tactics (`hax_mvcgen`), and turns `#[hax_lib::requires]`/`#[hax_lib::ensures]` into Lean `.pre`/`.post`/`.spec` definitions plus `sorry` theorem stubs in a generated `ProofObligations.lean` [B1; C1].
- Semantics of the generated spec (verbatim from B1):
  ```lean
  def barrett_reduce.spec (value : Std.I32) : Prop :=
    (barrett_reduce.pre value).holds →
    ⦃ ⌜ True ⌝ ⦄
    barrett_reduce value
    ⦃ ⇓ res => ⌜ (barrett_reduce.post value res).holds ⌝ ⦄
  ```
  "The specification states that, if the precondition holds, the function returns without failing and its result satisfies the postcondition." Code is in the `RustM` monad, "which models failures such as panics and integer overflow" [B1]. The triple uses Lean's `Std.Do` framework (`⦃ ⦄`, `⇓`, `mvcgen`), unlike Aeneas's own `spec` notation in REF-L07.
- "Contrarily to F*, successfully building the code doesn't prove panic freedom by default." [C1 `docs/manual/lean/quick_start.md`]

### 3.2 What is proved in Lean: libcrux-iot SHA-3 (provencrypt)
- Spec: `pedantic-sha3`, an AI-transcribed Rust version of FIPS 202 Algorithms 1-11 (bit state array), tested against FIPS properties, NIST CAVP vectors and other implementations [B2; C2]. Unbounded bitstrings: `BitStr` and `Nat` Rust wrappers are excluded from extraction and modelled in Lean as `List Bool` and `Nat` [B2].
- Contracts (Rust, verbatim, `annotated-upstream/src/lib.rs` via C2 README):
  ```rust
  #[hax_lib::ensures(|out| out.declassify()[..]
      == pedantic_sha3::bytes::shake128(data.declassify_ref(), BYTES)[..])]
  pub fn shake128<const BYTES: usize>(data: &[U8]) -> [U8; BYTES]

  #[hax_lib::requires(digest.len() == SHA3_256_DIGEST_SIZE)]
  #[hax_lib::ensures(|_| future(digest).declassify_ref()
          == &pedantic_sha3::bytes::sha3_256(payload.declassify_ref())[..])]
  pub fn sha256_ema(digest: &mut [U8], payload: &[U8])
  ```
- Extracted Lean statement (`proofs/LibcruxIotSha3/Extraction/Specs.lean:164`):
  ```lean
  def sha256.spec (payload : Slice Std.U8) : Prop :=
    ⦃ ⌜ True ⌝ ⦄
    sha256 payload
    ⦃ ⇓ res => ⌜ (sha256.post payload res).holds ⌝ ⦄
  ```
  discharged by `theorem sha256_spec_proof (payload : Slice Std.U8) : libcrux_iot_sha3.sha256.spec payload` (`Verification/ProofObligations.lean:442`). Thirteen such contracts: `hash`, `sha224`, `sha256`, `sha384`, `sha512`, `shake128`, `shake256`, and the `_ema` variant of each except `hash` [C2 README table].
- Intermediate lemma (not trusted), verbatim from C2 README: `theorem keccakf1600_equiv_lanes (s : state.KeccakState) (h_i : s.i = 0#usize) : ⦃ ⌜ True ⌝ ⦄ keccak.keccakf1600 s ⦃ ⇓ r_impl => ⌜ keccakFLanes (Permutation.lift s) = Permutation.lift r_impl ⌝ ⦄`. Proof meets in the middle at a pure-Lean "lane model"; both halves are proved [C2].

### 3.3 What is proved in F* (libcrux main)
- ML-KEM: verification status table in `libcrux-ml-kem/proofs/verification_status.md` (script-generated; "treat the table below as a rough guide"). Examples: generic `ind_cca` 27/27 panic-free and 26/27 correct; `ind_cpa` 0/21; `sampling` 0/5; `matrix` 1/5 panic-free and 0/5 correct; portable arithmetic/NTT/serialize/compress all verified; AVX2 mostly verified; Neon 0/13 arithmetic, 0/7 NTT, 0/7 compress, 0/12 serialize [C3].
- ML-DSA: "The portable and AVX2 code for field arithmetic, NTT polynomial arithmetic, and serialization is formally verified using hax and F*." [C3 `libcrux-ml-dsa/README.md:9-10`]
- Lean content in libcrux main: only `specs/sha3/proofs/aeneas-lean` (987 lines, a Lean extraction of a hacspec-style SHA-3 spec with no proofs; 0 `sorry`/`axiom` hits), and an empty `crates/algorithms/aes/proofs/lean/extraction/` with a `.gitignore` [C3]. The SHA-3 spec README says its Lean prerequisite hax-evit is "not publicly available yet" [C3 `specs/sha3/Readme.md`].
- Lean ML-KEM: B2 mentions "our work on the verification of ML-KEM, which builds on top of SHA-3" using Claude Fable 5; no released Lean ML-KEM artefact was found.

## 4. Property classes covered
For the Lean work (libcrux-iot SHA-3, C2); F* coverage in parentheses where different.
| Class | Status | Evidence |
|---|---|---|
| Functional correctness | yes | 13 one-shot contracts equal to `pedantic-sha3` (F*: ML-KEM/ML-DSA modules per status table). |
| Safety invariant | partial | Panic-freedom on contracted functions via `⦃ ⦄ ... ⦃ ⇓ ⦄`; memory safety inherited from safe Rust. |
| Liveness | no | Not addressed. |
| Refinement | yes | Implementation equal to the Rust spec, via an intermediate Lean lane model. |
| Symbolic security | no | (hax has a ProVerif backend; not used in these Lean sources.) |
| Computational security | no | Not in the Lean sources read. See CatCrypt lead in section 15. |
| Parsing | no | No wire formats in the Lean work. |
| Constant-time | no | "We only verify functional correctness here: the Lean verification does not contain any claims about absence of side channel vulnerabilities." [C2]; secret independence enforced only by Rust types (`libcrux_secrets`), modelled in Lean "as identities and no-ops" [C2]. |
| Other | no | |

## 5. Link to implementation code
Mechanism: extraction (Rust source with in-code contracts, translated via Charon/Aeneas into Lean, proofs written against the extraction). Guarantee: the Lean theorems hold for the hax/Aeneas model of the extracted Rust items and the extracted Rust spec, assuming hax's Rust-core models. The verified code is a snapshot of libcrux-iot at commit `5df472a` with three annotation patches, not the live libcrux-iot crate; patch 0003 "drops the `debug_assert!`s of the `u32::MAX` bound on the payload", and patch 0001 drops F*-oriented annotations, "the contracts on internal functions and on the incremental API" [C2]. B2: "In a few cases, we had to change the implementation itself to make the Lean extraction work", e.g. splitting 400+ line functions. Spec and implementation are separate crates, and "the plumbing between implementation and specification crates in Lean is done by hand" because hax lacks multi-crate extraction [B2].

## 6. Trusted computing base and soundness hygiene
hax Lean library (`hax-lib/proof-libs/lean`, 36 files, 23,009 lines, `rg -w`): `sorry` 0, `admit` 0, `axiom` declarations 0, `native_decide` 0, `implemented_by` 0, `extern` 0, `opaque` 2 (one docstring, one test), `partial` 1 (`meta partial def haxMvcgenLoop`, a tactic). It inherits the Aeneas fork's backend (see REF-L07 section 6 for Aeneas main counts).

provencrypt SHA-3 proofs (`sha3/libcrux-iot-5df472a/proofs`, 68 files, 26,526 lines): `sorry` 17 hits, of which 13 are the generated stubs in `Extraction/ProofObligations.lean`, which the lakefile excludes from the build ("sorry-filled, @[spec]-tagged, and imported by nothing ... OUT of the build"); the other 4 are in docstrings. `axiom` declarations 0 in the proofs; the spec package's `Extraction/*External_Template.lean` files contain template axioms (e.g. `axiom bits.BitStr : Type`), templates not used directly (hand-written models live in `Assumptions/`). `native_decide` 5 hits, `bv_decide` 5 hits (not classified line by line). Axiom guards, verbatim from `Verification/ProofObligations.lean:603-607`:
```lean
/--
info: 'libcrux_iot_sha3.Verification.shake128_spec_proof' depends on axioms: [propext, Classical.choice, Quot.sound]
-/
#guard_msgs in
#print axioms shake128_spec_proof
```
The README says all main theorems use only these three axioms, "checked on every build by `#guard_msgs` guards" [C2].

Unverified translators and other trust: rustc; "charon and aeneas (Lean backend) ... Both are trusted"; hax's modifications to Aeneas and `hax-lib` macro expansion; Rust-written core models, their extraction, and backend-native primitives; Lean and F* checkers [C1 trust.md]. hax 0.4 release (Cryspen blog): "Hax 0.4 targets Lean v4.31.0, which has some known soundness bugs. Upgrading the Lean version is planned for the next release." [B5, search excerpt]. comparator not yet run: "This is currently blocked on a Lean upgrade." [B2]

## 7. Scale, effort, automation
- Lean SHA-3 proof: "around 15,700 lines long, excluding comments and blank lines"; "The initial proof development took around a week"; "developed almost entirely by AI, with limited human guidance"; models named: Claude Opus 4.7 and Claude Fable 5 [B2]. The AI "mostly bypassed" `mvcgen` and "opened up the monad ... worked directly with tactics like `rw`, `simp`, `omega`, and `decide`" [B2].
- Raw size at C2 commit: 26,526 Lean lines in the proof package including the generated extraction.
- Maintenance: "From our work on verifying libcrux with F*, we have noticed that F* proofs are much harder to maintain, even with AI." [B2]
- Build time: not reported in the sources read.

## 8. Limitations stated by the authors (verbatim)
- "For now, hax only targets safe Rust, and can only be used to prove functional correctness and security properties. It cannot guard against side channel attacks, verify unsafe Rust code or model concurrency. Wrappers around Rust code that are written in other languages are out of scope, even though they could introduce bugs." [C1 trust.md]
- "hax doesn't support all Rust constructs, e.g, `unsafe` code or interior mutability." [C1 quick_start.md]
- "Running the prover (e.g., `lake build`) may not always run all the proofs that are supposed to be run. ... the Lean theorem might be in a file that is not actually imported and as a result remains unchecked when running `lake build`." [C1 trust.md]
- "Not all top-level API functions have been verified. In particular, we did not verify the incremental API (either buffered or unbuffered), and we did not verify the `Digest`/`Hasher` implementations. Also, secret independence is only ensured through Rust type mechanisms, not through Lean." [B2]
- "hax does not currently support multi-crate extractions"; "The hax model of Rust core is still missing a few items that we need in our SHA-3 specification. We currently model them ourselves." [B2]
- "Currently, hax's `hax_lib::implies` predicate does not support lazy evaluation." [B2] (The hax CHANGELOG "Unreleased" section now adds `implies!` with lazy right-hand side and notes "The Rust semantics of `implies` is no longer reversed" [C1].)
- "Not all the code in libcrux is formally verified yet." "the verification guarantees for libcrux code only extends to the verified modules." [B4]
- libcrux: "executables compiled from the code in this repository are *not* verified to be side-channel resistant" [C3 Readme.md:65].

## 9. Independent evidence
- Kobeissi, "Verification Theatre" (IACR ePrint 2026/192, abstract only): "we present thirteen vulnerabilities that escaped formal verification. Nine reside in unverified code ... Four reside in formally verified specification and proof code: in ML-KEM, a wrong decompression constant, a missing inverse NTT, and a false serialization proof; in ML-DSA, a wrong multiplication specification that renders axiomatized AVX2 proofs unsound." [B6]. An earlier version says the false proof "survives because the build system admits proofs by default" [B7]. These concern the F* development, not the Lean work.
- Microsoft's SymCrypt paper benchmarks libcrux (hax/F*-verified) as slower than SymCrypt Rust on SHA-3 (about 15%) and AES-GCM (6-12×), and HPKE (5.6× seal) [REF-L07 A1 §8.3]. This is a competitor's measurement.
- No independent review of the Lean SHA-3 proof was found.

## 10. Reproducibility
Not built: sandbox disk limit; queued for CI reproducibility job. Exact commands (from C2 and B2):
```bash
git clone https://github.com/cryspen/provencrypt && cd provencrypt
cargo install cargo-run-bin && cargo bin --sync-aliases     # pins cargo-hax 0.4.2
cargo hax extract pedantic-sha3 && cargo hax extract libcrux-iot-sha3
git diff --exit-code                                         # extraction up to date
cd sha3/libcrux-iot-5df472a && make check                    # patches reproduce annotated-upstream
cd proofs && lake exe cache get && lake build               # runs #guard_msgs axiom guards
```
Static inspection only, via `git clone --depth 1` at the commits in section 1.

## 11. Generalisation
Pattern: contracts written in Rust (`requires`/`ensures`) next to the code, an executable Rust transcription of the standard as specification, Rust-written models of core, extraction via Charon/Aeneas to Lean, AI-written untrusted proofs, axiom guards in CI.
To apply to an arbitrary codebase: safe Rust only (no unsafe, interior mutability, concurrency; same Charon/Aeneas subset as REF-L07, plus hax-specific gaps such as missing core models, no multi-crate extraction, `derive(Debug)`/formatting issues); the specification must itself be expressible as Rust that hax can extract; code may need refactoring (large functions split).
Case-specific preconditions: SHA-3 is sequential, deterministic, single-threaded, with a precise standard; the spec could be transcribed line by line. Only one-shot APIs were verified; stateful incremental APIs were not. No protocol, I/O, network adversary, or concurrency was modelled in the Lean work.

## 12. Bearing on sub-claims (facts only, no verdict)
| Sub-claim | Fact from this reference | Source ID |
|---|---|---|
| H1 | Specification is an executable Rust transcription of FIPS 202, extracted to Lean; unbounded bitstrings modelled via excluded Rust types mapped to `List Bool`/`Nat`. No protocol model in the Lean work. | B2, C2 |
| H2 | 13 one-shot SHA-3/SHAKE contracts proved in Lean (about 15.7k proof lines, about 1 week, mostly AI). Larger libcrux proofs (ML-KEM, ML-DSA modules) are in F*, with incomplete module coverage. | B2, C2, C3 |
| H3 | Link is by extraction from Rust source, but the verified artefact is a patched snapshot (`5df472a` plus 3 patches) rather than the live crate. Charon, Aeneas (modified), hax macros and Rust-core models are trusted. | C1, C2 |
| H4 | Rust-only, safe subset; unsafe, interior mutability and concurrency unsupported. | C1 |
| H5 | No end-to-end protocol result in Lean; hax's protocol-level tool is a separate ProVerif backend. | C1 |
| D | "Verification" per hax = "the parts of the Rust code that were extracted satisfy their specification, assuming ..."; libcrux status distinguishes "Lax Checking", "Runtime Safety" (panic-free) and "Correctness" (which can mean "some math properties", ranges, or full spec). | C1, C3 |

## 13. Conflicts
1. Lean backend status. hax README (main) labels Lean (via Aeneas) "Recommended for Lean" and the 0.4 blog makes it the main backend [C1, B1]; the third-party catalogue sorryfree.com says "with a Lean backend in development" and "Lean backend not yet complete" (status dated 2026-09-15, before the 0.4 release) [search excerpt, not used as evidence].
2. Toolchain versions. The hax 0.4 Cryspen blog says it "targets Lean v4.31.0" [B5]; hax `main` proof library is on `v4.35.0` [C1]; provencrypt pins `hax-lean v0.3.30` and Lean `v4.31.0` while its Cargo pins hax 0.4.2 per B2 [C2]. Probably different release lines (`hax-lean` versioning versus `cargo-hax`), not resolved.
3. Repository homes. hax docs link `github.com/pereq-dev/hax` and the Lean library requires `pereq-dev/aeneas` [C1]; provencrypt uses `cryspen/hax-lean` and `cryspen/aeneas` [C2]; the clone of `cryspen/hax` used here resolves through the same project. libcrux docs and A1 cite `celabshq/libcrux` while `cryspen/libcrux` also resolves [C3]. Organisational rename ("A new name for the next chapter", cryspen.com, 2026-10-22, search excerpt only) not investigated.
4. `implies` semantics: B2 (2026-10-12) says `implies` is not lazy; hax CHANGELOG Unreleased adds a lazy `implies!` and says the Rust semantics of `implies` "is no longer reversed" [C1], indicating a bug in the prior version.

## 14. Sources
| ID | Title | URL | P/S | Depth | Summary |
|---|---|---|---|---|---|
| B1 | hax 0.4: easier to install and configure, and a new Lean backend (hax blog, 2026-09-29), read from `docs/blog/posts/hax-0.4-release.md` in C1 | https://hax.cryspen.com/blog/2026/09/29/hax-04-easier-to-install-and-configure-and-a-new-lean-backend/ | P | FULL | Lean backend now Charon+Aeneas; spec shape; `sorry` stubs; scenarios; tool pinning. |
| B2 | Developer-driven Rust verification using hax, Lean, and AI (Cryspen, A. Bentkamp, 2026-10-12) | https://cryspen.com/post/sha3-hax-0-4/ | P | FULL | libcrux-iot SHA-3 Lean verification; specs in Rust; AI proofs; limitations. |
| B3 | provencrypt README / libcrux-iot README via C2 | https://github.com/cryspen/provencrypt | P | FULL | See C2. |
| B4 | The strengths and limits of formal verification (Cryspen, 2026-10-03) | https://cryspen.com/post/strengths-and-limitations/ | P | search excerpt | Libcrux verified modules only; experimenting with Lean instead of F*. |
| B5 | Hax 0.4! (Cryspen, 2026-10-03) | https://cryspen.com/post/hax_0_4_release/ | P | search excerpt | Pivot to Charon+Aeneas in early 2026; Lean v4.31.0 known soundness bugs. |
| B6 | Verification Theatre: False Assurance in Formally Verified Cryptographic Libraries (Kobeissi, ePrint 2026/192) | https://eprint.iacr.org/2026/192 | S | abstract only | 13 vulnerabilities in libcrux/hpke-rs, 4 inside verified F* spec/proof code. |
| B7 | Semantic Scholar entry for B6 (earlier version abstract) | https://www.semanticscholar.org/paper/Verification-Theatre:-False-Assurance-in-Formally-Kobeissi/ddcd38802e39467fca4814dbc0d703414cdf91e2 | S | abstract only | Earlier abstract: false serialization proof survives as build admits proofs by default. |
| C1 | cryspen/hax @ 4d52f78 (README, CHANGELOG, docs/manual/faq/trust.md, docs/manual/lean/*, hax-lib/proof-libs/lean) | https://github.com/cryspen/hax | P | CODE | Backends table, TCB statement, Lean library hygiene. |
| C2 | cryspen/provencrypt @ 9f7380e (sha3/spec, sha3/libcrux-iot-5df472a) | https://github.com/cryspen/provencrypt | P | CODE | Contracts, extracted specs, proofs, axiom guards, patches, lakefile. |
| C3 | cryspen/libcrux @ 1f9c898 (Readme.md, libcrux-ml-kem, libcrux-ml-dsa, specs/sha3, proofs/fstar) | https://github.com/cryspen/libcrux | P | CODE | F* status table; Lean content limited to a spec extraction. |

## 15. Unverified leads (excluded from evidence)
- CatCrypt: "From Rust to Cryptographic Security in Lean" (ePrint 2026/604): claims an "end-to-end pipeline from Rust reference implementations to security proofs in the computational model in Lean", Rust-to-Lean "done using the Hax tool", with ML-KEM IND-CCA and binding properties. Search excerpt only: https://eprint.iacr.asia/2026/604.pdf
- `cryspen/libcrux-proofs` repository (search result; appears to contain `proofs/fstar`); not inspected.
- `cryspen/hax-lean` ("Hax Lean library (automatically generated from cryspen/hax)") and the private `cryspen/hax-evit`.
- Cryspen ML-KEM Lean verification in progress (B2 mention); no artefact located.
- flyingpenguin.com claim that "only 58.4% of ML-KEM deployment code actually has its proofs checked"; secondary blog, not checked.
- Topic file 02 cites a "This Month in Hax: February 2026" post on `grind`/`bv_decide` attribute options; not read.
