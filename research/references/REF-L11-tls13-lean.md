# REF-L11: tls13-lean (plus pg-lean, same author)

## 1. Identity
| Field | Value |
|---|---|
| Subject (protocol or algorithm) | TLS 1.3 client and server (record layer, handshake codecs, sans-I/O state machines, key schedule, X.509 path validation) over HACL* C primitives. Companion: pg-lean, a PostgreSQL wire-protocol client (with SCRAM-SHA-256 and its own TLS 1.3 client path) that depends on tls13-lean. |
| Spec it targets (e.g. RFC 8446 section) | README: "TLS 1.3 for Lean 4 implementing RFC 9846" (RFC 9846 obsoletes RFC 8446); key-schedule spec is a transcription of RFC 8446 §7.1 (with §7.2/§7.3 traffic keys), X.509 per RFC 5280-style path validation. pg-lean: PostgreSQL frontend/backend protocol v3, SCRAM-SHA-256 (RFC 5802/7677), libpq `sslmode` semantics. |
| Organisation / authors | GitHub org `pb64-lean`; all commits authored "Bill Peterson <petersonbill64@gmail.com>" (89 of 89 in tls13-lean, 98 of 98 in pg-lean). No institutional affiliation stated. |
| Primary URLs | https://github.com/pb64-lean/tls13-lean ; https://github.com/pb64-lean/pg-lean ; org page https://github.com/pb64-lean |
| Repo, commit SHA, last commit date, licence | tls13-lean `0c61f21f3f0e46aa8789cd10f4e03871d1c002db`, 2026-09-09T18:20:03-04:00, Apache-2.0. pg-lean `6d1d1043cd82bf64e8dc504892bd4b4a7081ff35`, 2026-09-10T17:36:57-04:00, Apache-2.0 (pins tls13-lean at rev `0c61f21…` in `lake-manifest.json`). |
| Lean toolchain (or other tool and version) | tls13-lean `leanprover/lean4-nightly:nightly-2026-04-25` (no Lake dependencies; core + Std only). pg-lean `leanprover/lean4:v4.31.0`. Authoritative build is Bazel 8.5 with `rules_lean`; the `lakefile.lean` is described as an "IDE project model". |
| Maturity (research prototype / production / abandoned / AI-generated) | research (single-author engineering project with extensive tests and CI; not peer-reviewed; no paper). AI provenance: `commit-trailers` (see §9). |

## 2. Claim as stated by the authors (verbatim)
From tls13-lean `README.md` (commit `0c61f21`):

> "TLS 1.3 for Lean 4 implementing RFC 9846: pure-Lean protocol machinery — record layer, handshake codecs, sans-I/O **client and server** state machines, and X.509 path validation — over **formally verified, constant-time crypto primitives** bound from HACL\* via an explicit C FFI."

> "**Verification status, stated precisely.** Two distinct things are machine-checked, and it matters which is which:
> - The imported **HACL\* C primitives** carry externally machine-verified correctness and constant-time proofs. This repository does not re-prove them; it binds them across an explicit FFI boundary and trusts them.
> - The **Lean protocol code** carries kernel-checked laws about the implementation itself — the record framer (byte conservation, fragmentation independence, sequence/nonce injectivity, seal/open inversion parametric over the opaque AEAD), the handshake codecs (wire roundtrips with explicit residual, body inversion, GREASE-tolerant extension preservation, ClientHello canonicity …), the **state machines** (finite write traces over the actual `(AEAD key, nonce)` pairs …), the **key schedule** (structural refinement against RFC 8446 §7.1's derivation tree …), and the DER decoder … These are proofs about the executable definitions, not a parallel model."

> "What is **not** claimed: no security proof of the handshake or state machines, no cryptographic correctness of HKDF, the AEAD, the hashes, or the signature primitives (they are opaque FFI bindings, trusted from HACL\*), no timing analysis of Lean control code, and nothing about the C shims beyond their length preconditions." (README.md:158-162)

From pg-lean `README.md` heading (line 261): "## Proved properties (kernel-checked, no `sorry`, core + Std only)" and "`policy_no_insecure_fallback` proves no encryption-requiring mode may retry in a weaker transport."

## 3. What is actually formalised
Protocol modelling style: the implementation is written directly in Lean as pure "sans-I/O" state machines (`State`, `Op`, `Output`, `feed`, `run`). There is no separate abstract protocol model, no adversary model and no network/environment model; theorems are stated about the executable functions. Cryptographic primitives are `@[extern] opaque` constants (16 of them, `HaclStar/*.lean`), so nothing about their outputs can be proved; theorems that need primitive behaviour take it as a hypothesis or are parametric over an abstract `Spec.Hkdf`.

Key statements (tls13-lean @ `0c61f21`):

AEAD key/nonce non-reuse for a whole client run — `Tls/Client/Laws.lean:1984` (server mirror at `Tls/Server/Laws.lean:1907`):
```lean
theorem run_nonce_nodup {ops : List Op} {state : State} {out : Output}
    (h : run state ops = .ok out) :
    ∃ (aeadKeys : List ByteArray) (keyNonces : List (ByteArray × ByteArray)),
      Record.AeadWriteRun state.writeKeys? out.state.writeKeys? aeadKeys keyNonces ∧
        (aeadKeys.Nodup → keyNonces.Nodup)
```
Note the hypothesis `aeadKeys.Nodup` (cross-epoch key distinctness) is left to the caller; the README says it "is not a deterministic theorem about the opaque HACL\* binding".

Core lemma — `Tls/Record/Laws.lean:1689`:
```lean
theorem AeadWriteRun.nodup {src dst : Option TrafficKeys}
    {aeadKeys : List ByteArray} {keyNonces : List (ByteArray × ByteArray)}
    (h : AeadWriteRun src dst aeadKeys keyNonces)
    (hfresh : aeadKeys.Nodup) : keyNonces.Nodup
```

Seal/open inversion, conditional on AEAD correctness supplied as hypothesis `haead` — `Tls/Record/Laws.lean:1274`:
```lean
theorem open_seal
    (haead : ∀ key nonce aad plaintext,
      HaclStar.ChaCha20Poly1305.decrypt key nonce aad
          (HaclStar.ChaCha20Poly1305.encrypt key nonce aad plaintext) =
        some plaintext)
    {keys next : TrafficKeys} {contentType : ContentType}
    {fragment wire : ByteArray} {paddingLength : Nat}
    (h : «seal» keys contentType fragment paddingLength = .ok (next, wire)) :
    ∃ ciphertext : ByteArray,
      wire = encodeHeader .applicationData legacyRecordVersion
          ciphertext.size ++ ciphertext ∧
      RawRecord.WellFormed ⟨.applicationData, legacyRecordVersion, ciphertext⟩ ∧
      «open» keys ⟨.applicationData, legacyRecordVersion, ciphertext⟩ =
        .ok (next, { contentType, fragment, paddingLength })
```

Key-schedule structural refinement, parametric over any `Spec.Hkdf` that the bindings `Implements` — `TLS13/KeySchedule/Refinement.lean:122`:
```lean
theorem masterSecret_spec {H : Spec.Hkdf} (hi : Implements H) (inp : Spec.Inputs) :
    masterSecret (Spec.handshakeSecret H inp) (inp.hash .empty) =
      Spec.masterSecret H inp
```
The only unconditional byte-level key-schedule theorem — `TLS13/KeySchedule/Refinement.lean:70`:
```lean
theorem hkdfLabel_bytes (length : Nat) (label context : ByteArray) :
    hkdfLabel length label context =
      ByteArray.mk #[UInt8.ofNat (length >>> 8), UInt8.ofNat length,
          UInt8.ofNat (6 + label.size), 0x74, 0x6c, 0x73, 0x31, 0x33, 0x20] ++
        label ++ ByteArray.mk #[UInt8.ofNat context.size] ++ context
```
The RFC §7.1 derivation diagram is encoded as data and checked as `Derived.tree_rfc8446` (`TLS13/KeySchedule/Spec.lean:306`). Engine linkage: `Tls.Client.feed_keySchedule` (`Tls/Client/Laws.lean:3713`), `completeServerHandshake_verified` (`Tls/Client/Laws.lean:1750`), `run_wellFormed` (`Tls/Client/Laws.lean:1205`, server `Tls/Server/Laws.lean:1170`).

Codec canonicity: `parseClientHello_canonical` (`Tls/Handshake.lean:5638`). DER: `decode_tbs_encoded` (`TLS13/X509/Certificate.lean:927`).

pg-lean (@ `6d1d104`) examples:
```lean
-- Pg/Config.lean:111
theorem policy_no_insecure_fallback (m : SslMode) :
    (m.policy).requireEncryption = true → (m.policy).allowsFallback = false
```
```lean
-- Pg/Protocol/Machine.lean:3436
theorem terminal_pops_head {s : State} {pipe : Pipeline} {msg : RawMessage}
    {s' : State} {ev : Event} {out : ByteArray} {k : OpKind} {rest : List OpKind}
    (hph : s.phase = .running pipe) (hwf : pipe.WellFormed)
    (hfifo : pipe.pending = k :: rest) (hterm : isTerminal k ev = true)
    (h : step s msg = .ok (s', #[ev], out)) :
    ∃ pipe', s'.phase = .running pipe' ∧ pipe'.pending = rest
```
```lean
-- Pg/Protocol/Machine.lean:3680
theorem authStep_scram {c : Sasl.Scram.Client} {a : AuthState}
    (h : AuthStep (.scram c) a) : (∃ c', a = .scram c') ∨ a = .saslDone
```

## 3a. Predicate semantics
Headline 1: `Tls.Client.run_nonce_nodup` (`Tls/Client/Laws.lean:1984`). Conclusion predicate `Record.AeadWriteRun`, `Tls/Record/Laws.lean:1574-1594`:
```lean
inductive AeadWriteRun : Option TrafficKeys → Option TrafficKeys →
    List ByteArray → List (ByteArray × ByteArray) → Prop where
  | idle : AeadWriteRun none none [] []
  | done (keys : TrafficKeys) :
      AeadWriteRun (some keys) (some keys) [keys.key] []
  | protect {keys next : TrafficKeys} {dst : Option TrafficKeys}
      {contentType : ContentType} {fragment wire nonce : ByteArray}
      {paddingLength : Nat} {aeadKeys : List ByteArray}
      {keyNonces : List (ByteArray × ByteArray)}
      (hnonce : keys.nonce = .ok nonce)
      (hseal : «seal» keys contentType fragment paddingLength = .ok (next, wire))
      (hrest : AeadWriteRun (some next) dst aeadKeys keyNonces) :
      AeadWriteRun (some keys) dst aeadKeys ((keys.key, nonce) :: keyNonces)
  | rekey … : AeadWriteRun (some keys) dst (keys.key :: aeadKeys) keyNonces
  | install … : AeadWriteRun none dst aeadKeys keyNonces
```
Plain meaning: any successful client run (`run state ops = .ok out`) is witnessed by a trace listing every AEAD key the run used (`aeadKeys`) and every (key, nonce) pair it passed to `seal` (`keyNonces`), and if the keys in the trace are pairwise distinct then no (key, nonce) pair repeats. Strongest premise: `aeadKeys.Nodup` — distinctness of the HKDF-derived traffic keys across epochs is not proved (HKDF is opaque) and is left to the caller. Guaranteed: the Lean record layer never reuses a nonce under one key within a run. Not guaranteed: that keys are distinct, that AEAD is secure, or anything about the read direction.

Headline 2: `open_seal` (`Tls/Record/Laws.lean:1274`, §3). Strongest premise: `haead`, the full ChaCha20-Poly1305 decrypt-after-encrypt identity for the opaque HACL\* externs, taken as a hypothesis. Conclusion: the wire bytes are header ++ ciphertext, the record is well formed, and `open` on that record returns the original fragment, content type and padding with the same next-state keys. Guaranteed: Lean framing, padding and nonce/sequence handling invert correctly assuming AEAD correctness. Not guaranteed: AEAD correctness itself.

Headline 3: `masterSecret_spec` (`TLS13/KeySchedule/Refinement.lean:122`). Premise `Implements H`, `TLS13/KeySchedule/Refinement.lean:34-41`:
```lean
structure Implements (H : Spec.Hkdf) : Prop where
  hashLen_eq : H.hashLen = hashLen
  extract_eq : ∀ salt ikm, H.extract salt ikm = HaclStar.Hkdf.extractSha256 salt ikm
  expand_eq : ∀ prk info len,
    H.expand prk info len = HaclStar.Hkdf.expandSha256 prk info (UInt32.ofNat len)
```
The file notes that the HACL\* instance `hacl` (line 46) inhabits `Implements`, "so none of them is vacuous". Plain meaning: the engine's master-secret computation equals the RFC 8446 §7.1 derivation tree (as transcribed in `TLS13/KeySchedule/Spec.lean`) evaluated with whatever HKDF the bindings are; it fixes the wiring of labels, contexts and transcript hashes, not the bytes HKDF produces.

Headline 4 (pg-lean): `policy_no_insecure_fallback` (`Pg/Config.lean:111`) concludes `(m.policy).allowsFallback = false` from `(m.policy).requireEncryption = true`: a property of the configuration table, not of network behaviour.

## 4. Property classes covered
| Class | Status | Evidence |
|---|---|---|
| Functional correctness | partial | Codec roundtrips and canonicity (`parseClientHello_canonical`), seal/open inversion conditional on AEAD hypothesis (`open_seal`), DER slice retention; no end-to-end "implements RFC 9846" theorem. |
| Safety invariant | proved (implementation-level) | `run_wellFormed`, `State.WellFormed` preservation, nonce non-reuse within an epoch (`run_nonce_nodup`), pg-lean FIFO attribution (`terminal_pops_head`), `policy_no_insecure_fallback`. |
| Liveness | no | Only local progress lemmas in pg-lean (`step_progress`: an expected reply always steps); no temporal/liveness properties. |
| Refinement | partial | Structural refinement of the key schedule against a Lean transcription of RFC 8446 §7.1, parametric over HKDF (`masterSecret_spec`, `Derived.tree_rfc8446`); pg-lean `step_startup_order` refines a documented auth sequence. No refinement of the full handshake against an RFC state machine. |
| Symbolic security | no | README: "no security proof of the handshake or state machines". |
| Computational security | no | Same quote; primitives opaque. |
| Parsing | proved | Record framing byte conservation and fragmentation independence, handshake codecs, DER decoder, pg-lean framing partition invariance. |
| Constant-time | assumed (primitives only) | Inherited from HACL\* for primitives only; README: "no timing analysis of Lean control code"; pg-lean "Lean protocol/control code makes no timing guarantees". |

## 5. Link to implementation code
impl_link=lean-is-impl (impl_links: lean-is-impl, differential-testing). Lean is the implementation. The theorems are about the same Lean definitions that are compiled and run (README: "These are proofs about the executable definitions, not a parallel model"). The cryptography is C (HACL\* portable C, pinned commit) behind 16 `@[extern]` bindings plus two hand-written C shims (`HaclStar/shim/hacl_shim.c` 285 lines, `signature_shim.c` 118 lines); the Lean side cannot see into them. Conformance evidence beyond proofs: interop with OpenSSL `s_client`, curl and Go `crypto/tls` via a scripted loopback gate, RFC 8448 and KAT vectors (README "Tests"). pg-lean is also Lean-as-implementation, live-tested against PostgreSQL 17/18. Guarantee: properties hold of the compiled Lean code modulo the Lean compiler/runtime, the C shims and HACL\*, and the explicit theorem hypotheses (AEAD correctness, key distinctness, single-threaded state use).

## 6. Trusted computing base and soundness hygiene
Counts (whole-word grep over all non-`.lake` `.lean` files, comment lines starting `--` excluded; prose in `/-! -/` docstrings is not excluded, so raw counts overstate):

| Repo | Lean files / LOC | sorry | admit | axiom | native_decide | implemented_by | extern | opaque | partial |
|---|---|---|---|---|---|---|---|---|---|
| tls13-lean | 43 / 27,106 | 0 | 0 | 1 raw, 0 declarations (prose) | 0 | 0 | 18 raw = 16 `@[extern]` attributes + 2 in comments | 34 raw, 18 `opaque` declarations | 9 raw, 1 `partial def` (in `Test/TlsExternalServer.lean:70`) |
| pg-lean | 62 / 28,042 | 0 | 1 raw (prose) | 1 raw (prose) | 0 | 3 (`Pg/Protocol/Machine.lean:1425,1434`, `Pg/Protocol/Frontend.lean:248`) | 0 | 9 raw | 33 raw, 14 `partial def` (4 in `Pg/`) |

- Axiom policy is machine-audited in Bazel: `assurance.bzl` allows only `propext`, `Classical.choice`, `Quot.sound`; README: "Nothing depends on `sorryAx`, on `Lean.ofReduceBool`/`ofReduceNat`, or on a generated axiom of any kind", and "no `@[extern]` constant may live outside the `HaclStar` modules".
- pg-lean's three `implemented_by` attributes replace the logical definition with a faster "candidate" at runtime. Each has a private equality theorem (e.g. `submitAllCandidate_eq_submitArray`, `Pg/Protocol/Machine.lean:1367`; `bindCandidate_eq_reference`, `Pg/Protocol/Frontend.lean:233`), but the compiler's use of `implemented_by` is itself trusted (the kernel never checks that the runtime function is the one proved equal).
- README TCB list (tls13-lean): "the two C shims; the fifteen pinned HACL\* translation units …; the Lean compiler and runtime, including the `Nat` big-integer backend used by the pure-Lean RSA verifier …; and the Bazel/Nix toolchain pins … there is no zeroization of Lean-side key material."
- Explicit caller obligations: single-threading the `State` ("Lean values are duplicable, and no theorem about a pure function can forbid copying one").

## 7. Scale, effort, automation
- tls13-lean LOC by directory: `HaclStar` 269, `TLS13` 4,971, `Tls` 17,935, `Test` 3,888; largest files `Tls/Handshake.lean` 6,432, `Tls/Client/Laws.lean` 3,733, `Tls/Server/Laws.lean` 2,717, `Tls/Record/Laws.lean` 2,096. Implementation and proofs are interleaved; the `*/Laws.lean` files (8,546 lines) are proof-only. 749 `theorem` declarations. The Bazel assurance targets audit 23 (TLS13) and 88 (Tls) "principal theorems".
- pg-lean LOC: `Pg` 16,021, `Integration` 4,454, `Test` 7,368, `Cmd` 150; 494 `theorem` declarations.
- Effort: history shows tls13-lean was created 2026-08-02 with an initial commit already containing 35 `.lean` files, and 73 of 89 commits are dated 2026-08-04; the last commit is 2026-09-09. Total visible history is ~5 weeks for 27k lines with proofs. No person-effort figure is given by the author.
- Automation: standard Lean tactics (`omega`, `simp`, `fun_cases`, `decide`); README states byte identities are proved "arithmetically rather than by `bv_decide`". No ATP/SMT oracle.

## 8. Limitations stated by the authors (verbatim)
- "What is **not** claimed: no security proof of the handshake or state machines, no cryptographic correctness of HKDF, the AEAD, the hashes, or the signature primitives (they are opaque FFI bindings, trusted from HACL\*), no timing analysis of Lean control code, and nothing about the C shims beyond their length preconditions."
- "It is **structural**, not cryptographic. HKDF-Extract and HKDF-Expand are opaque `@[extern]` HACL\* bindings, so no theorem here can say the schedule produces the right *bytes*."
- "`aeadKeys.Nodup → keyNonces.Nodup` is an explicit theorem hypothesis. … it is not a deterministic theorem about the opaque HACL\* binding."
- "It covers the **write** direction only."
- "Unsupported: PSK and session resumption …, 0-RTT, the exporter and resumption secrets, post-handshake client authentication, AES-GCM suites, client-side HelloRetryRequest processing, and post-quantum hybrid groups".
- "`Record.Decoder.feed` has an explicit closure limitation: it frames a whole input chunk before the state machine processes its records."
- pg-lean: "|Timing side channels|Mixed|HACL\* primitives are constant-time; Lean protocol/control code makes no timing guarantees|"; "|TLS 1.3 encryption|Partial| … PostgreSQL SSLRequest path only (not PG 17+ direct negotiation)|"; "CRL/OCSP revocation is not implemented".

## 9. Independent evidence
- No paper, audit, CVE or independent review found. Adoption: a fork exists on GitHub user `keithadler`'s profile (search-result listing only, see §15); the author's own ecosystem (`grpc-lean`, `lean-acme-widgets`, `lean-pgx`) depends on it.
- Interop evidence is self-reported (OpenSSL, curl, Go `crypto/tls` handshakes; PostgreSQL 17/18 live tests).
- AI provenance (Conventions rule 7): category `commit-trailers`. Basis: in pg-lean, 2 of 98 commits (`b485891`, `336afde`, 2026-08-29) carry `Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>` and `Claude-Session: https://claude.ai/code/session_01JRjgcMANSQ7ZusVvqVQnSW`. In tls13-lean, 0 of 89 commits carry an AI trailer; the README makes no statement about AI use. Recorded commit-metadata facts, without characterisation: both repos share an identical "Initial commit" timestamp (2026-08-02T18:53:02-04:00); the tls13-lean initial commit contained 71 files (35 `.lean`); 73 of 89 tls13-lean commits are dated 2026-08-04. These facts do not establish who or what wrote the proofs; the kernel check is independent of authorship.

## 10. Reproducibility
Not built: sandbox disk limit; queued for CI reproducibility job. A first `lake build` attempt in `/tmp/tls13-lean` failed during toolchain installation with `No space left on device (os error 28)` while extracting `lean-4.31.0-nightly-2026-04-25`; per coordinator instruction it was not retried and the partial toolchain was deleted. Exact commands for the CI job:
```
git clone --depth 1 https://github.com/pb64-lean/tls13-lean && cd tls13-lean && git checkout 0c61f21f3f0e46aa8789cd10f4e03871d1c002db
lake build            # type-checks HaclStar, TLS13, Tls (no C link; IDE model)
bazel test //...      # authoritative: links HACL* shim, runs 10 suites + 3 lean_assurance_test audits (needs sibling ../grpc-lean, Bazel 8.5)
git clone --depth 1 https://github.com/pb64-lean/pg-lean && cd pg-lean && lake build
```
The repo's CI badges (`ci.yml`, `assurance.yml`) are self-reported evidence that these pass upstream.

## 11. Generalisation
Pattern: "own the protocol logic, borrow the primitives" — write the protocol engine in Lean as a pure sans-I/O state machine, prove implementation-level laws (codec roundtrip, framing invariance, nonce non-reuse, structural key-schedule refinement against an RFC transcription), and bind verified C crypto through a small audited FFI boundary whose correctness is passed as theorem hypotheses. A machine-checked axiom/extern audit in the build makes the TCB claim checkable.
To apply to an arbitrary codebase: this pattern requires rewriting the protocol logic in Lean (it does not verify existing C/Rust/Go TLS code). For an existing implementation one would need either extraction (REF-L07/L08) or differential/trace conformance testing against the Lean engine. Case-specific preconditions: the protocol logic is deterministic and pure once I/O and randomness are factored out; primitives exist in an already-verified library with a C ABI; the deployer accepts the Lean compiler/runtime in the TCB.

## 12. Bearing on sub-claims (facts only, no verdict)
| Sub-claim | Fact from this reference | Source ID |
|---|---|---|
| H1 | TLS 1.3 client/server state machines, record layer and RFC 8446 §7.1 key-schedule diagram are expressed in Lean; no adversary, network or probability model is expressed. | S1, S3 |
| H2 | 749 kernel-checked theorems over 27k LOC, including whole-run nonce non-reuse and structural key-schedule refinement, with zero `sorry` and an audited three-axiom closure; no symbolic or computational security property is proved. | S1, S3 |
| H3 | Theorems are about the executed Lean code (Lean-as-implementation); link to C is an FFI trust boundary with AEAD/HKDF behaviour as hypotheses; interop with OpenSSL/curl/Go is tested, not proved. | S1, S3 |
| H4 | Only applies to protocol code written in Lean; existing implementations in other languages are not covered. | S1 |
| H5 | Implementation properties (nonce non-reuse, parsing, refinement of the key-schedule shape) are proved without any protocol-level security proof — the README explicitly disclaims the latter. | S1 |
| D | Author uses "machine-checked" / "kernel-checked laws" for Lean proofs, "formally verified" for HACL\*, and "verify" in the X.509 / signature sense; "structural refinement" is distinguished from cryptographic correctness. | S1 |

## 13. Conflicts
- pg-lean README (local clone at 6d1d104) states the client inventory contains "no `opaque`"; static grep finds 9 raw `opaque` tokens and 3 `implemented_by` attributes in `Pg/` (the latter replace kernel-checked definitions at runtime). The README's statement concerns `opaque` declarations and `@[extern]`; it does not list `implemented_by` in its inventory. Not resolved here (raw counts include prose).
- README headline says "implementing RFC 9846" while the key-schedule spec and identifiers are RFC 8446 §7.1; README explains RFC 9846 obsoletes 8446 with unchanged labels. Recorded, not resolved.

## 14. Sources
| ID | Title | URL | P/S | Depth | Summary |
|---|---|---|---|---|---|
| S1 | tls13-lean README.md @ 0c61f21 (accessed 2026-11-07) | https://github.com/pb64-lean/tls13-lean | P | [CODE] | Full statement of scope, verification status, TCB, assurance audit, protocol scope and limitations. |
| S2 | pg-lean README.md @ 6d1d104 (accessed 2026-11-07) | https://github.com/pb64-lean/pg-lean | P | [CODE] | PostgreSQL client scope, proved-properties list, security compromises table. |
| S3 | tls13-lean source code @ 0c61f21 (accessed 2026-11-07) | https://github.com/pb64-lean/tls13-lean | P | [CODE] | Cloned depth 1; theorem statements, counts and `assurance.bzl` inspected. |
| S4 | pg-lean source code @ 6d1d104 (accessed 2026-11-07) | https://github.com/pb64-lean/pg-lean | P | [CODE] | Cloned depth 1; theorem statements, `implemented_by` sites and counts inspected. |
| S5 | tls13-lean and pg-lean full commit history (blobless clone, accessed 2026-11-07) | https://github.com/pb64-lean/tls13-lean/commits/main | P | [CODE] | 89 / 98 commits, single author; Claude co-author trailers in pg-lean; commit cadence. |

## 15. Unverified leads (excluded from evidence)
- GitHub search listing shows a fork of tls13-lean under user `keithadler` (search snippet only).
- `pb64-lean/lean-acme-widgets` README (search snippet) describes an end-to-end gRPC + PostgreSQL + TLS service using these libraries; not read in full.
- `docs/workload-identity.md` in tls13-lean (mTLS policy) was not read.
