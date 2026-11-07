# REF-C01: Project Everest (miTLS-fstar, HACL*, EverCrypt, EverParse)

Kind: comparator (non-Lean). Access date for all sources: 2026-11-07.

## 1. Identity
| Field | Value |
|---|---|
| Subject (protocol or algorithm) | TLS 1.2 / TLS 1.3 (record layer and handshake), QUIC record layer, cryptographic primitives (Curve25519, Ed25519, AES-GCM, ChaCha20-Poly1305, SHA-2/3, HMAC, HKDF, etc.), binary message formats (TLS wire formats, ASN.1 DER/X.509, Windows networking formats via 3D) |
| Spec it targets | TLS 1.3 drafts (record-layer paper interop-tested against draft 14 and draft 18 [C01-S2]; miTLS source comments link to `draft-ietf-tls-tls13` sections, e.g. `src/tls/MiTLS.Format.KeyShareEntry.fsti:13` [C01-S5]); message syntax transcribed from RFC presentation language into `src/parsers/MiTLS.Parsers.rfc` [C01-S5]; crypto algorithms against hand-written F* functional specs (`specs/Spec.*.fst`) [C01-S6] |
| Organisation / authors | Microsoft Research, INRIA, Carnegie Mellon University (21 authors on the 2026 retrospective incl. Bhargavan, Fournet, Hawblitzel, Parno, Protzenko, Swamy) [C01-S1] |
| Primary URLs | https://dl.acm.org/doi/full/10.1145/3805702 ; https://mitls.org/ ; https://github.com/project-everest/mitls-fstar ; https://github.com/hacl-star/hacl-star ; https://github.com/project-everest/everparse |
| Repo, commit SHA, last commit date, licence | mitls-fstar `9a75fa8c46e50897b1f43002885d25a34dfa929a`, 2026-07-16, Apache-2.0 ("Copyright 2015 INRIA and Microsoft Corporation"). hacl-star `f557259007db9933442dde10aa995c1b00bd3446`, 2026-10-29, Apache-2.0. everparse `8bd3c401bc8a6fbd29c5f21dd049116f83c84c15`, 2026-11-06, Apache-2.0. EverCrypt lives in hacl-star (`providers/`). |
| Tool and version | F* (Low* dialect) + Z3 SMT; KaRaMeL (F*-to-C), Vale (verified assembly). mitls-fstar pins F* as a git submodule `.fstar`. HACL* README: "HACL\* builds and verifies using a frozen F\* version, prior to the removal of the Low\* toolchain"; `flake.nix` pins `github:fstarlang/fstar?ref=fstar1` and KaRaMeL rev `bd2d4fa8a61344042c7cd7a31013d72bfe4568ce`. EverParse: `EVERPARSE_Z3_VERSION ?= 5.3.0` (`lib/everparse/z3-version.Makefile`). |
| Maturity | HACL*/EverCrypt: production (deployed code; repo now "in maintenance mode"). EverParse: production (deployed in Windows build). miTLS-fstar: research reference implementation; handshake proof incomplete; used historically in msquic, later superseded by Schannel [C01-S1]. Not AI-generated (multi-institution team since 2016). |

## 2. Claim as stated by the authors (verbatim)
- Retrospective goal list: "The TLS protocol, including support for both TLS-1.2 and TLS-1.3, covering both the record-layer and handshake subprotocols, with idealized code serving as a cryptographic specification." [C01-S1]
- Retrospective outcome: "Looking back, we accomplished much (but not all) of what we set out to do ... While we did offer fully verified, high-performance implementations of the record-layer protocols for TLS and QUIC, we did not complete verifying our *implementation* of the TLS handshake, though we did produce a detailed pen-and-paper proof of its cryptographic security, developing a novel technique for cryptographic proofs." [C01-S1]
- Record-layer paper abstract: "We build and verify a reference implementation of the TLS record layer and its cryptographic algorithms in F⋆ ... We reduce the high-level security of the record layer to cryptographic assumptions on its ciphers ... Combining our functional correctness and security results, we obtain the first verified implementations of the main TLS 1.3 record ciphers." [C01-S2]
- mitls.org (describing the F#/F7 stable version 0.9): "miTLS is a verified reference implementation of the TLS protocol ... We finally typecheck the protocol state machine, and thus obtain precise security theorems for TLS, as it is implemented and deployed." and "The development version is written and verified in F*" [C01-S3]
- HACL* README: "The code for all of these algorithms is formally verified using the F\* verification framework for memory safety, functional correctness, and secret independence (resistance to some types of timing side-channels)." [C01-S6]

## 3. What is actually formalised
Modelling style: code is written in Low* (a C-like F* subset); each function carries a pre/postcondition relating its memory effect to a pure F* specification; Z3 discharges verification conditions; KaRaMeL erases proofs and emits C. For protocols, "idealized code serving as a cryptographic specification" is used: modules are typed against ideal functionalities and a game-based security reduction is "verified by typing an F⋆ module" [C01-S2].

Example primitive theorem (HACL*, `code/curve25519/Hacl.Impl.Curve25519.Generic.fsti:13-23`, commit f557259):
```fstar
let scalarmult_st (s:field_spec) (p: Type0) =
    o:lbuffer uint8 32ul
  -> k:lbuffer uint8 32ul
  -> i:lbuffer uint8 32ul
  -> Stack unit
    (requires fun h0 ->
      p /\
      live h0 o /\ live h0 k /\ live h0 i /\
      disjoint o i /\ disjoint o k)
    (ensures  fun h0 _ h1 -> modifies (loc o) h0 h1 /\
      as_seq h1 o == S.scalarmult (as_seq h0 k) (as_seq h0 i))
```
`S.scalarmult` is the pure spec at `specs/Spec.Curve25519.fst:120` (`let scalarmult (k:scalar) (u:serialized_point) : Tot serialized_point =`).

Example of the protocol message layer (miTLS, `src/parsers/MiTLS.Parsers.rfc:1157-1164`): RFC presentation-language text, from which QuackyDucky/LowParse generate verified parsers ("Here we store the RFC specifications of all data formats, and we automatically generate their Low* specifications and implementations using QuackyDucky", `src/parsers/README.txt`):
```
struct {
  ProtocolVersion version;
  Random random;
  SessionID session_id;
  CipherSuite cipher_suites<2..2^16-2>;
  CompressionMethod compression_method<1..2^8-1>;
  /*@option*/ ClientHelloExtension extensions<2..2^16-1>;
 } ClientHello;
```

Handshake state machine: `src/tls/MiTLS.Handshake.fst` begins (line 24) with `#set-options "--admit_smt_queries true"`, repeated at lines 171, 300, 678, i.e. its SMT proof obligations are admitted, not checked [C01-S5]. Retrospective: "The proof of the TLS-1.3 handshake in miTLS-fstar was left incomplete" [C01-S1].

Adversary/environment: in the record-layer work the adversary "controls the TLS sub-protocols"; security is computational, with concrete bounds for AES-GCM and ChaCha20-Poly1305 [C01-S2]. Network below TLS is untrusted ("integrated below with untrusted networking layer (TCP, UDP, etc.)") [C01-S1].

## 4. Property classes covered
| Class | Status | Evidence |
|---|---|---|
| Functional correctness | yes (crypto primitives, parsers, record layer); partial (handshake) | Curve25519 postcondition above; retrospective statement on handshake [C01-S1, C01-S6] |
| Memory safety | yes for Low* code | HACL* README [C01-S6]; Low* dynamic frames [C01-S1] |
| Safety invariant (protocol state) | partial | handshake state machine typed but `--admit_smt_queries true` [C01-S5] |
| Liveness | no | not claimed in any source read |
| Refinement | yes, Low* code refines high-level F* spec ("proving the code in Low* correct against a low-level functional specification; followed by a proof ... relating the low-level functional specification to a more abstract high-level functional specification") [C01-S1] |
| Symbolic security | no (not in Everest itself; ProVerif used later in Bert13, a post-Everest project) [C01-S1] |
| Computational security | yes for TLS/QUIC record layers ("proofs of security in the computational model for the TLS and QUIC record layers"); handshake only pen-and-paper SSP proof [C01-S1, C01-S2] |
| Parsing | yes: EverParse non-malleability ("we automatically proved the non-malleability of the wire formats for TLS-1.2 and TLS-1.3"), ASN.1 DER, 3D double-fetch freedom [C01-S1] |
| Constant-time | yes for HACL* (secret-independence via abstract secret integer types, pen-and-paper metatheorem) and Vale (certified taint analysis) [C01-S1] |

## 5. Link to implementation code
Mechanism: "F* is the implementation". Low* source is verified, then extracted by KaRaMeL to C (and Vale to assembly). Guarantee: the extracted C/asm is the code that was verified, modulo KaRaMeL (in the TCB: "as KaRaMeL is part of our TCB"), the C compiler, and Vale's "simple trusted printer" [C01-S1]. HACL*: "about 245,000 lines of F\* and Vale source files, and yields about 135,000 lines of verified C and assembly code" [C01-S1]. Deployed: "Mozilla Firefox, the Linux kernel, Python, mbedTLS, the Tezos blockchain" [C01-S1]. EverParse: a 3D/RFC-style format spec generates verified C parsers; for Windows Network Virtualization "around 30,000 lines of verified C code from about 5,000 lines of heavily commented" 3D specs [C01-S1]. Not applicable to pre-existing C code: the approach requires rewriting the implementation in F*.

## 6. Trusted computing base and soundness hygiene
Static counts, F* sources (`.fst`/`.fsti`), via `rg -c` at the commits above [C01-S5, C01-S6, C01-S7]:

| Repo | files | LOC | `admit()` | `assume` (word) | `assume val` | `admit_smt_queries true` | `--lax` |
|---|---|---|---|---|---|---|---|
| mitls-fstar (all) | 277 | 57,564 | 63 | 486 | 174 | 125 | 3 |
| hacl-star (code, specs, lib, vale) | n/a | 206,226 | 70 (57 of them in `lib/Lib.IntVector.fst`, a model of vector intrinsics that itself sets `--admit_smt_queries true`) | 121 | 96 | 2 | 1 |
| everparse (lib, src) | 706 files | n/c | 1 | n/c | 39 | 1 | n/c |

In mitls-fstar `src/tls`, 55 of 152 files contain `admit_smt_queries true`, `admit()` or `--lax`, including `MiTLS.Handshake.fst`, `MiTLS.Handshake.Secret.fst`, `MiTLS.HandshakeLog.fst`, `MiTLS.Negotiation.fst`, `MiTLS.StAE.fst`, `MiTLS.StreamAE.fst`, `MiTLS.Connection.fst`, `MiTLS.TLS.fst`, `MiTLS.QUIC.fst`. Files without these markers include `MiTLS.Record.fst`, `MiTLS.StatefulLHAE.fst`, `MiTLS.Alert.fst`. Also `src/tls/Makefile.common:147` sets `VFLAGS = --admit_smt_queries true` for imported library modules (FStar., LowStar., Hacl., Spec., Vale., Lib., LowParse.), i.e. they are re-used as pre-verified dependencies, not re-checked in the miTLS build. HACL*: `code/hpke/Hacl.HPKE.Interface.AEAD.fst:15-16` `let aead_encrypt #cs = admit()`; `specs/Spec.Agile.AEAD.fst:110` `admit()`.

Other TCB (authors): "F* lacks a kernel checker and also trusts both the encoding of VCs to the SMT solver and the SMT solver itself"; "In total, we have found 26 soundness bugs in F*, including three open issues that are undergoing repair"; "we still discovered a total of six bugs that were caught by neither our formalization, the internal KaRaMeL checker, nor the C compiler"; Vale relies on a "(trusted) deep embedding of each target hardware platform"; miTLS handshake "relied on an unverified region-based memory allocator"; "Low* has no support for concurrency" [C01-S1]. Record-layer paper lists admitted invariants: "in Crypto.Symmetric.PRF, we admit a simple integer-normalization invariant ... We also have five local assumptions related to memory allocation and write effects" [C01-S2].

## 7. Scale, effort, automation
- "a full build of Project Everest takes about 2 hours on a build server with 24 threads, building repositories with a total of about 1.3 million lines of non-auto-generated F* source text. The build issues around 66,000 SMT queries to automatically discharge around 622,000 separate proof obligations, producing around 4,200 checked F* files." [C01-S1]
- HACL*: "The entire library consists of 212,000 lines of F* code (specification and proof included), ultimately generating 101,000 lines of C code" (HACL* section) vs "about 245,000 lines of F* and Vale ... about 135,000 lines of verified C and assembly" (overview incl. Vale) [C01-S1].
- "more than 100,000 lines of code were written and verified in Low* by hand" [C01-S1]. Noise*: 59 protocol variants in "about 40,000 lines of generic, higher-order Low* and F* code" [C01-S1].
- Team: three institutions, five main locations, many interns/PhD students, from 2016 [C01-S1]. No person-year total located.
- Automation: SMT (Z3) dominant; metaprogramming/tactics; performance parity with unverified code (e.g. ADX Curve25519 "beat all known verified or unverified implementations"; Windows parsers within 2% overhead) [C01-S1].

## 8. Limitations stated by the authors (verbatim)
- "we did not complete verifying our *implementation* of the TLS handshake" [C01-S1]
- "due to a variety of factors, including the shifting priorities of a large team, we did not manage to translate this pen-and-paper proof to the verified code of miTLS-fstar. Nevertheless, miTLS-fstar is a complete implementation of TLS-1.3 and interoperates with other implementations." [C01-S1]
- "miTLS-fstar's (unverified) handshake can also be used independently for QUIC ... used by initial versions of WinQUIC" [C01-S1]
- "Most significantly, we did not tackle the HTTPS protocol itself." [C01-S1]
- "Low* has no support for concurrency. Our plan for reasoning about the concurrent uses of the top-level TLS interface of miTLS-fstar involved delegating the management of disjoint connections to unverified client code." [C01-S1]
- QUIC-in-Dafny effort: "a confluence of factors, including the pandemic ... led to this effort being abandoned." [C01-S1]
- Record-layer paper: "the record layer is part of a larger, partially-verified codebase—notably, we leave a complete verification of the TLS 1.3 handshake and its integration with our code as future work." [C01-S2]
- HACL* README: "HACL\* is in maintenance mode" [C01-S6].

## 9. Independent evidence
- Bugs found in standard: "work on miTLS-fstar helped uncover a few subtle bugs in the draft TLS-1.3 standard", e.g. version-downgrade susceptibility in early drafts [C01-S1]; mitls.org lists attacks (Triple Handshake, SMACK, FREAK, Logjam, SLOTH) from the earlier F#/F7 miTLS line [C01-S3].
- Adoption: HACL*/EverCrypt in Firefox, Linux kernel, Python, mbedTLS, Tezos; EverParse/F*/KaRaMeL/Z3 "available for use within the build of every Windows developer"; miTLS handshake used as default TLS handshake for msquic around 2018 until superseded by Schannel [C01-S1]. These are author-reported, not independent audits.
- Secondary/marketing claim: a sota.io blog (EU PaaS vendor) claims miTLS "has been verified against the formal TLS 1.3 specification (RFC 8446) and against computational security models" and that the "TLS state machine ... [is] verified against the TLS 1.3 RFC, and compiled to C code that runs in Firefox's NSS library" [C01-S4]. See section 13.

## 10. Reproducibility
Not built: sandbox disk limit (instruction from /root: no F*/Z3/KaRaMeL installs). Commands that would be used: miTLS `git submodule update --init && make -C .fstar/src/ocaml-output && make -C src/tls` (per README); HACL* `nix develop` then `make` (per README). Static inspection only; clones deleted after extraction.

## 11. Generalisation
Pattern: "verified implementation in a proof-oriented language, extracted to C/asm" (correct-by-construction rewrite), plus "spec-as-RFC-grammar to verified parser generator" (EverParse/3D) and "idealised modules as cryptographic specification, security reduction by typing". To apply to an arbitrary codebase: the implementation must be rewritten in Low*/Vale (EverParse is the exception: it generates parsers that are linked into existing C code, only the parsing boundary is replaced). Case-specific preconditions: sequential code (no concurrency in Low*), SMT-friendly arithmetic, a team of proof experts, a trusted extraction tool (KaRaMeL), and an F* version freeze (HACL* builds only with frozen pre-removal Low* F*).

## 12. Bearing on sub-claims (facts only, no verdict)
| Sub-claim | Fact from this reference | Source ID |
|---|---|---|
| H1 | A non-Lean dependently typed language (F*) expressed TLS 1.3 record layer, handshake state machine (partly admitted), message formats, adversary-controlled sub-protocols and computational security games. | C01-S1, C01-S2, C01-S5 |
| H2 | Record-layer computational security and crypto functional correctness were machine-checked; handshake implementation proof was not completed by a large expert team; handshake key schedule was proved on paper only. | C01-S1, C01-S2 |
| H3 | Implementation link achieved by making the verified language the implementation and extracting to C; TCB includes KaRaMeL, Z3, F* (26 soundness bugs found), C compiler. | C01-S1 |
| H4 | Route requires rewriting code in Low*; existing C codebases are only reachable at parser boundaries (EverParse 3D). | C01-S1 |
| H5 | Parts of the stack (parsers, crypto, record layer) were verified without a completed protocol-level (handshake) proof; deployed handshake code was unverified. | C01-S1, C01-S5 |
| D | Authors distinguish "fully verified" (record layer), "pen-and-paper proof" (handshake security), and "(unverified)" (handshake for QUIC); mitls.org applies "verified reference implementation" to the whole of miTLS. | C01-S1, C01-S3 |

## 13. Conflicts
- miTLS and RFC 8446. Side A: sota.io blog [C01-S4] states "miTLS has been verified against the formal TLS 1.3 specification (RFC 8446) and against computational security models" and that the TLS state machine is "verified against the TLS 1.3 RFC". The mitls.org home page [C01-S3] calls miTLS "a verified reference implementation of the TLS protocol" (its detailed text refers to the F#/F7 0.9 release). Side B: the 2026 TOPLAS retrospective by the Everest authors [C01-S1] states "we did not complete verifying our *implementation* of the TLS handshake" and "The proof of the TLS-1.3 handshake in miTLS-fstar was left incomplete"; the 2016 record-layer paper [C01-S2] tested against TLS 1.3 draft 14/18 and left "a complete verification of the TLS 1.3 handshake ... as future work"; mitls-fstar source [C01-S5] sets `--admit_smt_queries true` in `MiTLS.Handshake.fst` and cites `draft-ietf-tls-tls13` rather than RFC 8446. Factual context, not a resolution: RFC 8446 was published in 2018, after the record-layer paper; [C01-S4] is a vendor blog that cites no source for the sentence; [C01-S1] covers the F* version (miTLS-fstar), whereas mitls.org's detailed security-theorem text describes the earlier F# version.
- sota.io [C01-S4] says miTLS code "runs in Firefox's NSS library"; [C01-S1] names Firefox only among deployments of Everest crypto (HACL*/EverCrypt) and says miTLS handshake was deployed in msquic, later superseded by Schannel.
- HACL* size: 212,000 F* LOC / 101,000 C LOC (HACL* section) vs 245,000 F*+Vale LOC / 135,000 C+asm LOC (overview) within [C01-S1]; the difference is likely Vale inclusion, as worded, but recorded as given.

## 14. Sources
| ID | Title | URL | P/S | Depth | Summary |
|---|---|---|---|---|---|
| C01-S1 | Project Everest: Perspectives from Developing Industrial-Grade High-Assurance Software (ACM TOPLAS 48(2), published 2026-06-05) | https://dl.acm.org/doi/full/10.1145/3805702 | P | FULL (≈200k chars of article text; fetch truncated in the final reference list) | Authors' retrospective: what Everest verified (crypto, record layers, parsers), what it did not (handshake implementation, HTTPS), TCB, soundness bugs, deployments. |
| C01-S2 | Implementing and Proving the TLS 1.3 Record Layer (Bhargavan et al., ePrint 2016/1178, S&P 2017) | https://eprint.iacr.org/2016/1178.pdf | P | FULL | Verified F* record layer with computational security reduction, interop with draft-14/18 implementations; handshake verification left as future work. |
| C01-S3 | miTLS: A Verified Reference Implementation of TLS | https://mitls.org/ | P | FULL | Project home page; describes F#/F7 miTLS 0.9 security theorems and F* development version. |
| C01-S4 | Deploy F* to Europe — Verified Programming on EU Infrastructure in 2026 (sota.io blog) | https://www.sota.io/blog/deploy-fstar-europe-eu-hosting | S | FULL (read via extraction of all relevant sentences) | Vendor marketing post claiming miTLS verified against RFC 8446 and running in NSS; no citations for these claims. |
| C01-S5 | project-everest/mitls-fstar @ 9a75fa8 | https://github.com/project-everest/mitls-fstar | P | CODE | F* TLS implementation; handshake modules admit SMT queries; RFC-syntax parser specs. |
| C01-S6 | hacl-star/hacl-star @ f557259 | https://github.com/hacl-star/hacl-star | P | CODE | Verified crypto library (HACL*, Vale, EverCrypt); functional-correctness postconditions; maintenance mode. |
| C01-S7 | project-everest/everparse @ 8bd3c40 | https://github.com/project-everest/everparse | P | CODE | Verified parser generator (LowParse, QuackyDucky, 3D); pins Z3 5.3.0. |

## 15. Unverified leads (excluded from evidence)
- A search result listed "RFC 9846: The Transport Layer Security (TLS) Protocol Version 1.3" (rfc-editor.org), possibly an RFC 8446 successor; not read.
- Phase-1 topic files reference HAL-Inria copy of the record-layer paper and EverCrypt S&P 2020 paper; not separately read.
- SSProve / Bert13 (Rust + hax + SSProve + ProVerif) TLS 1.3 key-schedule work mentioned in [C01-S1]; primary sources not read.
