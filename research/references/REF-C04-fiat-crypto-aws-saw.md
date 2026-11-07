# REF-C04: Fiat-Crypto (Coq) plus AWS s2n-tls and aws-lc SAW proofs

Kind: comparator (non-Lean). Access date for all sources: 2026-11-07.

## 1. Identity
| Field | Value |
|---|---|
| Subject (protocol or algorithm) | Fiat-Crypto: finite-field arithmetic for elliptic curves (Curve25519, P-256, P-224, P-384, secp256k1 and others; "80 prime fields") [C04-S1]. s2n-tls: HMAC, DRBG, TLS 1.2 and TLS 1.3 handshake state machine, socket corking [C04-S2, C04-S3]. aws-lc: SHA-2 (384/512), HMAC-SHA-384, AES-KW(P), AES-GCM (plus earlier HKDF, ECDSA, ECDH entries now commented out) [C04-S4] |
| Spec it targets | Fiat-Crypto: Coq mathematical specs of field operations (e.g. `eval (carry_mul x y) mod m = (eval x * eval y) mod m`) [C04-S5]. s2n: Cryptol specs hand-compiled from RFCs 5246, 5077, 6066 (TLS 1.2) [C04-S2]; repo file `tests/saw/spec/handshake/rfc_handshake_tls13.cry` is described as "a specification of the TLS 1.3 handshake message state machine according to RFC 8446" [C04-S3]; HMAC against FIPS 198-1 [C04-S2]. aws-lc: Cryptol specs from Galois `cryptol-specs` [C04-S4]. |
| Organisation / authors | Fiat-Crypto: Erbsen, Philipoom, Gross, Sloan, Chlipala (MIT CSAIL) [C04-S1]. s2n SAW: Chudnov, Collins, Cook, Dodds, Huffman, MacCárthaigh, Magill, Mertens, Mullen, Tasiran, Tomb, Westbrook (Galois, Amazon, UCL, Univ. of Washington) [C04-S2]. aws-lc-verification: AWS / Galois [C04-S4]. |
| Primary URLs | http://adam.chlipala.net/papers/FiatCryptoSP19/FiatCryptoSP19.pdf ; https://discovery.ucl.ac.uk/id/eprint/10075271/1/Chudnov2018_Chapter_ContinuousFormalVerificationOf.pdf ; https://github.com/aws/s2n-tls ; https://github.com/awslabs/aws-lc-verification ; https://github.com/mit-plv/fiat-crypto |
| Repo, commit SHA, last commit date, licence | mit-plv/fiat-crypto `1e721c6f18c43b90922e7bc57fd64848e5cff2be`, 2026-10-19, MIT OR Apache-2.0 OR BSD-1-Clause. aws/s2n-tls `d695290a8e4c7107bb2c5afa359a46c0624a1c50`, 2026-11-06, Apache-2.0. awslabs/aws-lc-verification `797302470c1703cbb1a3a6fa98daffef0bfd3d80`, 2026-03-26, Apache-2.0. |
| Tool and version | Fiat-Crypto: Coq/Rocq (`coq-fiat-crypto-without-bedrock.opam`: `"coq" {>= "9.0~"}`, `"rocq-stdlib" {>= "9.1~"}`), plus Bedrock2 option. s2n: SAW binary `saw-0.9.0.99-Linux-x86_64` downloaded by `codebuild/bin/install_saw.sh`; Cryptol; Z3 via `w4_unint_z3`. aws-lc: SAW + Cryptol, NSym (AArch64), Coq for spec validation; verified code compiled with Clang 10 (C) and Clang 10/14 (AArch64 asm) [C04-S4]. |
| Maturity | Production: Fiat-Crypto output in BoringSSL ("Chrome, Android, and CloudFlare") [C04-S1]; s2n-tls proofs in CI of a production AWS library [C04-S2]; aws-lc is FIPS 140-3 certified [C04-S6].|
| AI provenance | commit-trailers. Basis: GitHub API, last 100 commits per repo, checked 2026-11-07. mit-plv/fiat-crypto has 39/100 commits with Claude co-author trailers (2026-05-28 to 2026-10-19). aws/s2n-tls has 1/100 (2026-11-06). awslabs/aws-lc-verification has 0/100. No agent-config files at the repo roots. The S&P 2019 and CAV 2018 results predate these commits. Recorded as provenance only, not as a discount on the mathematics. |
| Evidence level | deployed (Fiat-generated code in BoringSSL and others [C04-S1]; s2n SAW proofs run in AWS CI [C04-S2]); proof content: static-inspected |

## 2. Claim as stated by the authors (verbatim)
- Fiat-Crypto: "We introduce a new approach for implementing cryptographic arithmetic in short high-level code with machine-checked proofs of functional correctness ... As one outcome, we present the first verified high-performance implementation of P-256, the most widely used elliptic curve." [C04-S1]
- s2n: "we built a proof and associated infrastructure for s2n's implementations of DRBG, HMAC, and the TLS handshake. The proof targets an existing implementation" and "we have proved that (1) it implements a subset of TLS 1.2 as defined in IETF RFCs 5246, 5077 and 6066 and (2) the socket corking API ... is used correctly. Formally, we proved that the implementation refines a specification (conversely, the specification simulates the implementation)." [C04-S2]
- aws-lc-verification README: "Only a subset of algorithms are formally verified, and only for certain platforms." [C04-S4]
- AWS (NIST workshop paper): "For the most commonly used and recommended algorithms, parameter sets and key lengths, we aim for verification of full functional correctness with respect to a formal specification" [C04-S6].

## 3. What is actually formalised
Fiat-Crypto: arithmetic algorithms written once, generically, in Gallina over lists of limbs; partial evaluation plus a certified compiler (bounds analysis, word-size selection) specialise them for a given prime and word size; output is straight-line code pretty-printed to C, Rust, Go, Java, Zig, Bedrock2 [C04-S1, C04-S5]. Spec (`src/COperationSpecifications.v:278-284`, commit 1e721c6):
```coq
    Definition carry_mul_correct
               (carry_mul : list Z -> list Z -> list Z)
      := forall x y,
        list_Z_bounded_by loose_bounds x
        -> list_Z_bounded_by loose_bounds y
        -> eval (carry_mul x y) mod m = (Z.mul (eval x) (eval y)) mod m
           /\ list_Z_bounded_by tight_bounds (carry_mul x y).
```
proved for each synthesized program (`src/PushButtonSynthesis/UnsaturatedSolinas.v:743-746`: `Lemma carry_mul_correct res (Hres : carry_mul = Success res) : carry_mul_correct ... (Interp res). Proof using curve_good. prove_correctness (). Qed.`). Generated C carries the spec as a comment, e.g. `fiat-c/src/curve25519_64.c:127-131` "Postconditions: eval out1 mod m = (eval arg1 * eval arg2) mod m".

s2n-tls (SAW): existing C compiled to LLVM bitcode, symbolically executed by SAW, and checked against Cryptol specs. HMAC: a chain monolithic Cryptol spec → incremental Cryptol spec → C, with the high-level spec linked to a Coq security proof of HMAC by Beringer et al. [C04-S2]. Handshake: Cryptol "RFC specification" with `handshakeTransition : Parameters -> State -> State`, proved to simulate a Cryptol model of s2n, which is proved equivalent to the C state machine [C04-S2]. TLS 1.3 property in the current repo (`tests/saw/spec/handshake/rfc_handshake_tls13.cry:36-41`, commit d695290):
```cryptol
tls13rfcSimulatesS2N : {len} (fin len, len >= 1) => connection -> Parameters -> Bit
tls13rfcSimulatesS2N conn params =
    (initial_connection conn /\ connectionParameters conn params) ==>
        tracesMatchExcludingWaits
            (map rfc2S2N (doHandshake`{len} params))
            (traceS2N`{len} conn)
```
checked at `tests/saw/spec/handshake/handshake.saw:109-110` with `prove_print (w4_unint_z3 []) {{ tls13rfcSimulatesS2N `{16} }};` i.e. for traces of length 16.

aws-lc: SAW `llvm_verify` / `llvm_verify_x86` proofs of EVP API operations against Cryptol specs for fixed platforms, with listed caveats (section 6) [C04-S4].

Adversary: none modelled for the handshake: "We assume that the TLS handshake as specified in the RFCs is secure, and do not formalize nor verify any cryptographic properties of the specification" [C04-S2]. HMAC high-level spec has a pseudorandom-function security proof (Coq, prior work) [C04-S2].

## 3a. Predicate semantics

Fiat-Crypto field multiplication, `src/COperationSpecifications.v:278-284` @ 1e721c6 [C04-S5]:
```
    Definition carry_mul_correct
               (carry_mul : list Z -> list Z -> list Z)
      := forall x y,
        list_Z_bounded_by loose_bounds x
        -> list_Z_bounded_by loose_bounds y
        -> eval (carry_mul x y) mod m = (Z.mul (eval x) (eval y)) mod m
           /\ list_Z_bounded_by tight_bounds (carry_mul x y).
```
Here `eval` is `Positional.eval wt n` (line 253: `Local Notation eval := (Positional.eval wt n).`), the weighted sum of limbs. `list_Z_bounded_by` (lines 47-50) is `is_bounded_by bounds v = true` (each limb within its `zrange`).

Plain language (Fiat). Guaranteed: on inputs whose limbs are within the loose bounds, the generated function computes the field product modulo `m` and returns limbs within the tight bounds. This holds for all such inputs, not only for sampled ones. The statement is about the Coq-level function. Its transfer to the emitted C relies on Fiat's verified pipeline down to its own C-like AST and on the trusted pretty-printer and C compiler (§5, §6). Not guaranteed: behaviour on out-of-bounds inputs; constant-time execution of the compiled binary; and protocol-level properties.

s2n-tls TLS 1.3 state machine, `tests/saw/spec/handshake/rfc_handshake_tls13.cry:36-41` @ d695290 [C04-S3]:
```
tls13rfcSimulatesS2N : {len} (fin len, len >= 1) => connection -> Parameters -> Bit
tls13rfcSimulatesS2N conn params =
    (initial_connection conn /\ connectionParameters conn params) ==>
        tracesMatchExcludingWaits
            (map rfc2S2N (doHandshake`{len} params))
            (traceS2N`{len} conn)
```
`tracesMatchExcludingWaits`, `tests/saw/spec/handshake/s2n_handshake_io.cry:203-209`:
```
// match actions, but drop any with an 'A' writer from the second trace
tracesMatchExcludingWaits: {n} (fin n, width n>=1) => [n]handshake_action -> [n]handshake_action -> Bit
tracesMatchExcludingWaits xs ys = go 0 0 where
  go: [width n] -> [width n] -> Bit
  go i j = if i >= `n then True
            | j >= `n then True
            else (xs@i == ys@j /\ go (i+1) (j+1))
```
(The comment says 'A' writers are dropped. The body as written advances `i` and `j` together and compares elementwise; this is recorded as observed, without interpretation.) Premise `initial_connection` (`s2n_handshake_io.cry:994` onward) requires `handshake_type < S2N_HANDSHAKES_COUNT`, `message_number == 0`, `corked` ∈ {0,1}, and TLS 1.3 consistency constraints (for example, early data accepted implies a non-null `chosen_psk`; HelloRetryRequest and early data are not both set; EarlyClientCCS implies MiddleboxCompat). It is proved only at `len = 16`: `prove_print (w4_unint_z3 []) {{ tls13rfcSimulatesS2N `{16} }};` (`handshake.saw:110`).

Plain language (s2n). Guaranteed: for every valid initial connection and matching parameters, the first 16 handshake actions of s2n's Cryptol state-machine model equal those of the Cryptol transcription of the RFC 8446 message flow. A separate SAW proof (`handshake_io_lowlevel.saw`) links the C functions `s2n_conn_set_handshake_type` / `s2n_advance_message` to `traceS2N`'s step function on LLVM bitcode. Not guaranteed: traces longer than 16; that the Cryptol RFC transcription is faithful to RFC 8446 (manual); message contents, cryptography or record processing (separate or absent proofs); and binary-level semantics beyond the LLVM bitcode that SAW reads. For aws-lc, the per-function SAW specs carry the README caveats quoted in §6 (limited input lengths, assumed malloc/free, `ToolGap`, and others).

## 4. Property classes covered
| Class | Status | Evidence |
|---|---|---|
| Functional correctness | yes (Fiat field ops; s2n HMAC/DRBG; aws-lc listed EVP ops, with caveats) | [C04-S1, C04-S2, C04-S4] |
| Safety invariant | yes for s2n handshake state machine (message sequence refinement), bounded to length 16 in current CI script | [C04-S2, C04-S3] |
| Liveness | no | |
| Refinement | yes: s2n "the implementation refines a specification"; Fiat: certified compilation from generic Gallina | [C04-S1, C04-S2] |
| Symbolic security | no | |
| Computational security | partial: HMAC spec linked to existing Coq PRF proof; none for TLS handshake | [C04-S2] |
| Parsing | no (not claimed) | |
| Constant-time | partial: Fiat "the language enforces the constant time security property by construction" at source level; compiler preservation presumed [C04-S1]. s2n uses SideTrail for constant-time (not SAW) [C04-S6] | |

## 5. Link to implementation code
Two distinct mechanisms.
- Fiat-Crypto: "synthesis/extraction" — Coq-certified compiler emits straight-line C (or other languages) that is then hand-integrated into libraries (BoringSSL, and aws-lc uses Fiat specs) [C04-S1, C04-S4]. Trusted: "All formal reasoning is done in the Coq proof assistant, and the overall trusted computing base also includes a simple pretty-printer and the C language toolchain" [C04-S1].
- SAW: "verification of existing code" — LLVM bitcode (and x86 machine code via `llvm_verify_x86`) of the production C library is symbolically executed against Cryptol specs; "In all cases, the actual verification is performed on code that is produced by Clang, but the verification results also apply to any compiler that produces semantically equivalent code" [C04-S4]. Proofs run in CI: "The proof is applied as part of the continuous integration system for s2n ... and runs every time a code change is" made [C04-S2]; current CI targets in `codebuild/bin/s2n_codebuild.sh:100-113` (`make -C tests/saw tmp/verify_HMAC.log tmp/verify_drbg.log failure-tests`, `make -C tests/saw tmp/verify_handshake.log`) [C04-S3].
This is the only reference in this batch that verifies pre-existing, unmodified-language production code (C) rather than code written in the verifier's language.

## 6. Trusted computing base and soundness hygiene
Static counts (`rg`), at commits above:

| Repo | files | LOC | soundness markers |
|---|---|---|---|
| fiat-crypto `src/**/*.v` | 679 | 183,411 | `Admitted.` 8, `admit.` 9, `Axiom` 163 (most indented inside module types, e.g. `src/Util/Structures/Orders.v`), `Parameter` 76, `Print Assumptions` 9. Top-level examples: `src/Curves/Weierstrass/P256.v:85` `Lemma p256_mul_mod_n ... Proof. Admitted.`; `src/Bedrock/P256.v:47-51` `Axiom p256_coord_sqr` / `p256_coord_mul_ok`; `src/Bedrock/P256/Platform.v:185,188` Admitted; `src/Curves/TableMult/TableMult.v:1228-1235` admits; `src/Arithmetic/ZmodSqrt.v:331-356` admits; `src/Assembly/Parse.v:578` `by admit`. Whether these lie in the dependency cone of the C-generation correctness lemmas was not determined (not built). |
| s2n-tls `tests/saw` | `.saw` 1,525 lines; `.cry` 2,959 lines | — | `crucible_llvm_verify`/`llvm_verify` 8 occurrences; `crucible_llvm_unsafe_assume_spec` 20 (12 in `verify_HMAC.saw` assuming hash functions, 6+ in `spec/handshake/handshake.saw` assuming socket cork/uncork, session-id generation, etc.); `enable_lax_loads_and_stores` noted in `handshake.saw`. `failure_tests/` holds patches (`tls_missing_full_handshake.patch`, `tls_early_ccs.patch`, `cork_one.patch`, `sha_bad_magic_mod.patch`) expected to make proofs fail. |
| aws-lc-verification `SAW/**/*.saw` | — | 15,181 | `llvm_verify*` 221; `llvm_unsafe_assume_spec` (incl. crucible_ prefix) 130; `admit` 0; `enable_lax` 2 |

aws-lc author-listed caveats (verbatim excerpts, README) [C04-S4]: "MemCorrect: Basic memory management functions such as OPENSSL_malloc and OPENSSL_free are not verified, and are assumed to behave correctly."; "InputLength: The implementation is verified correct only on a limited number of input lengths."; "SAWBreakpoint: ... The feature lacks well-foundedness check for the inductive invariant."; "GcmMultipleOf16: EVP_{Encrypt,Decrypt}Update are only verified for cases where whole blocks are encrypted/decrypted"; "GcmADNotVerified: Supplying additional data (AD) to AES-GCM is not verified."; "ToolGap: Adjacent components in the implementation are formally verified using different tools ... It is assumed that a proof of correctness in one tool implies correct behavior as determined by the other"; "SAWCore_Coq: Certain facts are admitted in Coq code provided by the saw-core-coq library."; "NoInline: ... assuming that certain functions are not inlined".

s2n: "SAW uses bounded symbolic execution"; "the implementation of s2n is small (less than 10k LOC), and most iteration is bounded" [C04-S2].

## 7. Scale, effort, automation
- Fiat-Crypto: 183,411 lines of Coq in `src` (static count) [C04-S5]; "competitive performance for 80 prime fields and multiple CPU architectures" [C04-S1]. No person-year figure located in [C04-S1].
- s2n SAW: "HMAC and DRBG each took roughly 3 months of engineering effort. The TLS handshake verification took longer at 8 months, though some of that time involved developing tool extensions"; "Since deployment of the proof to the CI system in November of 2016 our proofs have been re-played 956 times ... We have had to update the proof three times. In all cases the proof update was complete before the code review process finished." [C04-S2]
- aws-lc: SAW scripts 15,181 lines [C04-S4]. Automation: SAT/SMT (Z3, Yices/ABC via SAW), rewriting; Coq for spec-level facts.

## 8. Limitations stated by the authors (verbatim)
- Fiat: "we leave instruction selection, scheduling, and register allocation out of scope." [C04-S1]
- Fiat: "We do hope, in follow-on work, to prove that a particular compiler preserves the constant-time property in generated assembly. For now, we presume that popular compilers like GCC and Clang do preserve constant time" [C04-S1]
- Fiat: "The bounds-analysis pipeline described in this paper only works on straight-line code" and "We would like to shrink our trusted base by connecting to a certified compiler targeting assembly." [C04-S1]
- s2n: "We assume that the TLS handshake as specified in the RFCs is secure, and do not formalize nor verify any cryptographic properties of the specification." [C04-S2]
- s2n: TLS handshake proof covers "a subset of TLS 1.2" [C04-S2].
- aws-lc: "Only a subset of algorithms are formally verified, and only for certain platforms." and "Recently, we have changed our technical approach for formal verification to use tools newer than SAW and NSym." (s2n-bignum on HOL Light; CBMC for C) [C04-S4].

## 9. Independent evidence
- Bugs found: s2n "Our TLS Handshake proof revealed a bug (which was promptly fixed) in the s2n implementation" ("Missing branches in the handshake state machine", 2017) [C04-S2]; failure-test patch `tls_missing_full_handshake.patch` reproduces it [C04-S3].
- Adoption: Fiat code "shipped in Chrome 64"; BoringSSL developers "made use of our code-generation pipeline (without even consulting us, the tool authors)"; "about half of HTTPS connections opened by Web browsers worldwide use our fast verified code" (2019) [C04-S1]. aws-lc proofs "will be maintained and continue to run in CI" [C04-S4].
- AWS 2024 overview: s2n-tls "development and proof were carried out hand-in-hand, using Cryptol and SAW for functional correctness, CBMC for memory- and type-safety proofs, and SideTrail for verification of constant-time properties"; aws-lc AArch64 proof engines include "multiple SMT solvers and Lean4" [C04-S6].
- Fiat repo `crypto-defects.md` lists historic arithmetic bugs in other libraries (e.g. CVE-2014-3570 bignum squaring, openssl#1593 P384 reduction) as the class the verification targets [C04-S5].

## 10. Reproducibility
Not built: sandbox disk limit (instruction from /root: no Coq/SAW/Cryptol/Docker installs). Commands that would be used: Fiat `make standalone-ocaml` / `make` (opam `coq-fiat-crypto`); s2n `codebuild/bin/install_saw.sh <dl> <dir>` then `make -C tests/saw tmp/verify_HMAC.log tmp/verify_handshake.log failure-tests`; aws-lc `docker build --pull --no-cache -f Dockerfile.saw_x86 -t awslc-saw .` then ``docker run -v `pwd`:`pwd` -w `pwd` awslc-saw`` (README). Static inspection only; clones deleted after extraction.

## 11. Generalisation
Two patterns. (a) Fiat: "generic verified algorithm + certified specialising compiler → straight-line code in a mainstream language"; applies where code is straight-line arithmetic with a parameterisable mathematical spec; requires writing the algorithm in Gallina (CPS) and trusting a pretty-printer and C compiler. (b) SAW: "verify existing C/LLVM/x86 against an executable Cryptol spec by bounded symbolic execution, re-run in CI"; applies to existing code whose loops are bounded or can be handled with assumed inductive invariants (`SAWBreakpoint`), and whose callee functions can be specified and assumed (`llvm_unsafe_assume_spec`). Preconditions: fixed input lengths/platforms (caveats `InputLength`, platform table), hand-translated RFC specs, no cryptographic security for protocol logic.

## 12. Bearing on sub-claims (facts only, no verdict)
| Sub-claim | Fact from this reference | Source ID |
|---|---|---|
| H1 | An executable non-dependently-typed spec language (Cryptol) expressed TLS 1.2 and TLS 1.3 handshake message state machines hand-compiled from RFCs; no cryptographic security of the handshake was expressed. | C04-S2, C04-S3 |
| H2 | s2n handshake state-machine refinement took ~8 months; HMAC/DRBG ~3 months each; TLS 1.3 check bounded to length 16. | C04-S2, C04-S3 |
| H3 | Two links: Fiat certified compiler emits C; SAW checks existing Clang-produced LLVM/x86 code directly. | C04-S1, C04-S4 |
| H4 | SAW verifies existing production C (s2n-tls, aws-lc) with documented caveats (input lengths, assumed callee specs, unverified malloc). | C04-S2, C04-S4 |
| H5 | Components (field arithmetic, HMAC, state machine) were verified without a protocol-level security proof of TLS. | C04-S1, C04-S2 |
| D | "Verified" in aws-lc is qualified per algorithm/platform with named caveats; s2n "proved ... implements a subset of TLS 1.2". | C04-S2, C04-S4 |

## 13. Conflicts
- TLS version scope: CAV 2018 paper states the s2n handshake proof covers "a subset of TLS 1.2" [C04-S2]; the current s2n-tls repo contains a TLS 1.3 Cryptol spec "according to RFC 8446" and a SAW check `tls13rfcSimulatesS2N `{16}` [C04-S3]. Different dates (2018 paper vs 2026 commit); the repo check is bounded to 16-message traces.
- aws-lc scope: the 2024 AWS paper says AWS aims for "verification of full functional correctness" for commonly used algorithms [C04-S6]; the aws-lc-verification README lists a subset with caveats and has HKDF, ECDSA, ECDH, and EC key rows commented out of the "Verified Code" table [C04-S4]. Recorded as given; the README also states newer work moved to s2n-bignum, mlkem-native, mldsa-native (not read).

## 14. Sources
| ID | Title | URL | P/S | Depth | Summary |
|---|---|---|---|---|---|
| C04-S1 | Simple High-Level Code For Cryptographic Arithmetic – With Proofs, Without Compromises (Erbsen et al., IEEE S&P 2019) | http://adam.chlipala.net/papers/FiatCryptoSP19/FiatCryptoSP19.pdf | [P] | [FULL] | Fiat-Crypto: generic Coq arithmetic, partial evaluation, certified compiler to C; P-256 and Curve25519 in BoringSSL; TCB and limitations. |
| C04-S2 | Continuous Formal Verification of Amazon s2n (Chudnov et al., CAV 2018) | https://discovery.ucl.ac.uk/id/eprint/10075271/1/Chudnov2018_Chapter_ContinuousFormalVerificationOf.pdf | [P] | [FULL] | SAW/Cryptol proofs of s2n HMAC, DRBG, TLS 1.2 handshake state machine in CI; effort; bug found. |
| C04-S3 | aws/s2n-tls @ d695290 (`tests/saw`, `codebuild/`) | https://github.com/aws/s2n-tls | [P] | [CODE] | Current SAW scripts, Cryptol RFC specs for TLS 1.2/1.3 state machines, assumed specs, failure tests, CI targets, SAW 0.9.0.99 pin. |
| C04-S4 | awslabs/aws-lc-verification @ 7973024 (README and SAW scripts) | https://github.com/awslabs/aws-lc-verification | [P] | [CODE] | Verified-code table with caveats, platforms, tools; 221 verify / 130 unsafe-assume calls; move to s2n-bignum/CBMC. |
| C04-S5 | mit-plv/fiat-crypto @ 1e721c6 | https://github.com/mit-plv/fiat-crypto | [P] | [CODE] | Coq sources; operation specs; generated C with postconditions; admit/Axiom locations; crypto-defects list. |
| C04-S6 | Formal Verification of Cryptographic Software at AWS: Current Practices and Future Trends (Chapman et al., NIST workshop, April 2024) | https://www.nist.gov/system/files/documents/2024/06/11/01-ChapmanPHPLBK.pdf | [P] | [FULL] | AWS overview: aws-lc (SAW, Coq, NSym+Lean4, s2n-bignum HOL Light), s2n-tls (SAW, CBMC, SideTrail), s2n-quic (Kani), SampCert. |

## 15. Unverified leads (excluded from evidence)
- s2n-bignum (HOL Light), mlkem-native and mldsa-native (CBMC + HOL Light) named in [C04-S4]; not read.
- "Missing branches in the handshake state machine" (Chudnov, 2017 GitHub issue) cited in [C04-S2]; not opened.
- Fiat-Crypto follow-ups (Bedrock2 end-to-end, CryptOpt assembly equivalence) indicated by `src/Bedrock/End2End` and `src/Assembly` directories; papers not read.
- Beringer et al. Coq HMAC security proof (VST) referenced in [C04-S2]; not read.
