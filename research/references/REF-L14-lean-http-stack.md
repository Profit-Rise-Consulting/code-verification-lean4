# REF-L14: Lean HTTP stack: Std.Http (Lean core), eumemic/stdhttp-verify, AfonsoBitoque/LeanServer

This dossier covers three related artefacts:
- (A) `Std.Http`, the HTTP/1.1 server shipped in the Lean 4 standard library.
- (B) `stdhttp-verify`, a third-party project that tests `Std.Http` differentially and proves properties of a patched copy of it.
- (C) `LeanServer`, an independent HTTPS/TLS 1.3/HTTP/2/HTTP/3 server written in Lean 4.

They are kept apart throughout because their guarantees differ sharply.

## 1. Identity
| Field | Value |
|---|---|
| Subject (protocol or algorithm) | (A) HTTP/1.1 server protocol machine (sans-I/O) with chunked encoding and keep-alive. (B) Request framing, parsing and segmentation-independence of (A). (C) Full web server: TLS 1.3, HTTP/2 + HPACK, HTTP/3/QUIC, WebSocket, gRPC, pure-Lean AES/SHA-256/X25519. |
| Spec it targets (e.g. RFC 8446 section) | (A, B) RFC 9112 (§2.1, §3, §3.2, §5, §6.3, §7, §7.1), RFC 9110 §5.6.1. (C) RFC 8446, RFC 7541, RFC 9000/9114, RFC 7748, RFC 4648 (per `THREAT_MODEL.md`). |
| Organisation / authors | (A) Lean FRO; module headers read "Authors: Sofia Rodrigues" (all 47 files). (B) GitHub user `eumemic`. (C) Afonso Bitoque (sole committer in the shallow clone). |
| Primary URLs | (A) https://github.com/leanprover/lean4/tree/master/src/Std/Http. (B) https://github.com/eumemic/stdhttp-verify ; upstream issue https://github.com/leanprover/lean4/issues/15840 , PR https://github.com/leanprover/lean4/pull/15841. (C) https://github.com/AfonsoBitoque/LeanServer |
| Repo, commit SHA, last commit date, licence | (A) leanprover/lean4 @ `a2f1a642abafa8e133a408a75683bce5ff47d606`, 2026-11-06T18:18:48Z, Apache-2.0. (B) eumemic/stdhttp-verify @ `f19b903352bcde44e35672f02850d846093ca312`, 2026-10-29T02:29:35-07:00, Apache-2.0. (C) AfonsoBitoque/LeanServer @ `24b916aaa6ae4a20732536494904d0699fac7ec7`, 2026-05-14T16:56:49+01:00, Apache-2.0. |
| Lean toolchain (or other tool and version) | (A) master as above. (B) `spec/` and `fix/`: `leanprover/lean4:nightly-2026-10-20`; no Mathlib (`spec` depends only on `httpfix`). (C) `leanprover/lean4:v4.29.1`; no Mathlib; links OpenSSL and libgmp via C FFI (`src/crypto_ffi.c`, `lakefile.toml:26`). |
| Maturity (research prototype / production / abandoned / AI-generated) | (A) Shipped library code in Lean core; production-intent. (B) Research project, AI-assisted: `upstream/issue/body.md:82` "Disclosure: the testing, diagnosis and patch were done with AI assistance (Claude)."; `upstream/pr/body.md:9` "Developed with AI assistance (Claude)." (C) Hobby/research prototype; whether it was AI-generated cannot be determined from the shallow clone, as no README statement was found. Its documentation gives three different theorem totals (164, 914, 935; see §13), and some "theorems" are vacuous (see §6). |

## 2. Claim as stated by the authors (verbatim)
- (A) `src/Std/Http.lean` module doc: "A low-level HTTP/1.1 server implementation for Lean. This library provides a pure, sans-I/O protocol implementation that can be used with the `Async` library or with custom connection handlers." (A) makes no verification claim.
- (B) README: "Differential testing and formal verification of Lean's built-in HTTP/1.1 server, `Std.Http`." Also: "in a patched copy of `Std.Http` (`fix/`), the parsers the server runs on request syntax — request line, header lines, chunk-size lines, trailer sections — are rewritten as total, end-of-input-strict functions and proven to give the same outcome however the input is split into reads". And: "the grammar splits a byte stream in only one way, so a front end that divides the stream by the same grammar agrees with the server on every request it reads (`no_desync`)".
- (C) README (Portuguese): "Um servidor HTTPS completo implementado em **Lean 4**, com TLS 1.3, HTTP/2, HTTP/3 (QUIC), WebSocket, gRPC, e **935 teoremas formais** verificados pelo compilador — zero `sorry`. A especificação formal É o código executável: sem extracção, sem spec-impl gap." `THREAT_MODEL.md:11`: "LeanServer's 914 verified theorems (0 `sorry`, 0 `axiom`) provide the following guarantees".

## 3. What is actually formalised
### (A) Std.Http
It is an implementation, not a verification artefact: 47 files, 13,726 lines. The 25 theorem declarations are small helper lemmas, for example `Internal/Char.lean:103` `quotedStringChar_lt_0x80` and `Data/URI/Encoding.lean:220` `isValidUTF8_of_isAsciiByte`. The protocol machine's read/write loops are `partial def` (18 in `Protocol/H1.lean`, e.g. `:1319 partial def processWrite`, `:1759 partial def processRead`, `:1798 private partial def pullNextChunk`), so they cannot be unfolded in proofs. At `a2f1a64`, `Protocol/H1/Parser.lean:61` contains `def optionalUnlessEof`, the combinator introduced by PR #15841:
```lean
def optionalUnlessEof (p : Parser α) : Parser (Option α) := fun it =>
  match p it with
  | .success rem res => .success rem (some res)
  | .error rem .eof => .error rem .eof
  | .error _ _ => .success it none
```
### (B) stdhttp-verify
The model has four parts:
- A receive-loop abstraction (`parseWith`: re-run the parser after each read, `.eof` means wait).
- Byte-level RFC 9112 grammars as inductive relations (`Spec/Message.lean`: `HeadBytes`, `ChunkedBody`, `BodyOf`, `RequestBytes w h d`, `Requests w n`).
- The real `Std.Http` `Machine .receiving` from the patched copy, run under an arbitrary list of server operations `ops : List Op`.
- A hand-written model of the connection loop (`Spec/Real/Loop.lean`).

The environment is "any split of the byte stream into reads" plus any interleaving of server operations. The adversary is the byte stream, which is arbitrary. There is no cryptographic adversary. Key statements:

`spec/Spec/Segmentation.lean:77`:
```lean
theorem segmentationIndependent_iff {run : Run α} :
    (∀ buf pieces, feed run buf pieces = run (buf ++ pieces.flatten)) ↔ PrefixConsistent run
```
`spec/Spec/Real/Framing.lean:485` (stated against the shipped `getSize`; `check-shipped.sh` re-checks it against the toolchain's own `Std.Http`):
```lean
theorem getSize_assigns (m : Request.Head) {b : Body.Length}
    (h : Message.Head.getSize (dir := .receiving) m true = some b) :
    Assigns (m.version == .v10) (fieldValues m .transferEncoding) (fieldValues m .contentLength)
      (toBody b)
```
`spec/Spec/Real/Machine.lean:2118,2171`:
```lean
theorem reads_requests (cfg : Protocol.H1.Config) (ops : List Op) :
    let r := run ops { config := cfg }
    ∃ c, r.2 = c ++ rem r.1 ∧ Ok (ghostOf (pulls ops { config := cfg })) r.1 c
theorem nothing_after_rejection (cfg : Protocol.H1.Config) (ops₁ ops₂ : List Op)
    (hd : Dead (run ops₁ { config := cfg }).1) :
    Dead (run (ops₁ ++ ops₂) { config := cfg }).1 ∧
      (run (ops₁ ++ ops₂) { config := cfg }).2 = (run ops₁ { config := cfg }).2
```
`spec/Spec/Real/NoDesync.lean:108`:
```lean
theorem no_desync (cfg : Protocol.H1.Config) (ops : List Op) {ms : List (List UInt8)}
    {u : List UInt8} (hms : ∀ x ∈ ms, ∃ h d, RequestBytes x h d)
    (hs : (run ops { config := cfg }).2 = ms.flatten ++ u)
    (hr : Reading (run ops { config := cfg }).1) :
    let m := (run ops { config := cfg }).1
    ∃ (ps : List (List UInt8)) (q : List UInt8), (run ops { config := cfg }).2 = ps.flatten ++ q ++ rem m ∧
      (∀ x ∈ ps, ∃ h d, RequestBytes x h d) ∧ ps.length = m.reader.messageCount ∧
      (ps <+: ms ∨ ms <+: ps) ∧
      (HeadRead m → ∀ x ∈ ms[ps.length]?, ∀ h d, RequestBytes x h d →
        Recorded m.reader.messageHead h) ∧
      (m.reader.state = .complete → ∀ x ∈ ms[ps.length]?, q = x)
```
`spec/Spec/Real/Absorb.lean:2763`:
```lean
theorem pull_split (m : Machine .receiving) (x y : _root_.ByteArray) (hib : InBounds m)
    (hn : m.reader.noMoreInput = false) (hsm : m.writer.sentMessage = false) (hd : ¬ Dead m)
    (hbud : ChunkBudget m)
    (hbuf : (unread m.reader.input).length + x.size + y.size ≤ Machine.maxBufferedInputBytes m.config)
    (hnone : (m.feed x).pullBody.2 = none) :
    (m.feed (x ++ y)).pullBody.2 = ((m.feed x).pullBody.1.feed y).pullBody.2 ∧
      EquivE (m.feed (x ++ y)).pullBody.1 ((m.feed x).pullBody.1.feed y).pullBody.1
```
`spec/Spec/Real/LoopMain.lean:288`:
```lean
theorem serve_split (A : App) (cfg : Protocol.H1.Config) (eof : Bool) {S1 S2 : List _root_.ByteArray}
    (h1 : ∀ p ∈ S1, p.size ≠ 0) (h2 : ∀ p ∈ S2, p.size ≠ 0) (hf : flat S1 = flat S2)
    (o1 : NoOvf A (start cfg S1 eof)) (o2 : NoOvf A (start cfg S2 eof)) (r : List UInt8 × List Obs) :
    Serve A (start cfg S1 eof) r ↔ Serve A (start cfg S2 eof) r
```
`spec/README.md` documents further results: soundness and completeness per parser (`chunkSizeLine_sound`, `requestLine_complete`, `request_read`, `split_read`, `conn_read`, `conn_finish`), URI/Host acceptance (`targetHost_ok`), grammar uniqueness (`request_unique`, `requests_unique`) and `pulled_bodies`.

### (C) LeanServer
Spec modules (`LeanServer/Spec/*.lean`, 11 files, 5,454 lines) define an abstract TLS handshake state machine. They map it onto the implementation's `TLSState`. `LeanServer/Spec/TLSRefinement.lean:43,53,63`:
```lean
def implStateToModel : LeanServer.TLSState → HandshakeState
theorem impl_closed_maps_to_model_closed :
    implStateToModel .Closed = .Closed := rfl
theorem transitionToAppData_refines_model
    (session : TLSSessionTLS)
    (session' : TLSSessionTLS)
    (h : transitionToAppData session = some session')
    : implStateToModel session'.state = .Connected
```
Security "properties" rest on axioms. `LeanServer/Spec/TLSSpec.lean:275,293`:
```lean
axiom x25519_dh_commutativity :
  ∀ (a b : ByteArray),
  a.size = 32 → b.size = 32 →
  LeanServer.X25519.scalarmult a (LeanServer.X25519.scalarmult_base b) =
  LeanServer.X25519.scalarmult b (LeanServer.X25519.scalarmult_base a)
axiom ephemeral_key_independence :
  ∀ (k1 k2 : ByteArray),
  k1.size = 32 → k2.size = 32 → k1 ≠ k2 →
  LeanServer.X25519.scalarmult_base k1 ≠ LeanServer.X25519.scalarmult_base k2
```
The theorem `forward_secrecy_via_ephemeral_keys` (`TLSSpec.lean:325-330`) is proved by `exact ephemeral_key_independence k1 k2 h1 h2 hne`, which restates the axiom. `LeanServer/Server/Concurrency.lean:395,400,407`:
```lean
axiom io_ref_atomic_ops :
  ∀ (descr : String), descr = "IO.Ref operations are individually atomic per Lean 4 runtime"
axiom io_ref_compound_not_atomic :
  ∀ (descr : String), descr = "IO.Ref get-then-set is not atomic; TOCTOU possible"
theorem single_mutex_no_deadlock : (1 : Nat) * (1 - 1) = 0 := by omega
```
`LeanServer/Spec/ProtocolInvariants.lean:419` has `axiom gc_no_guaranteed_zeroization : ∀ (descr : String), descr = "Lean 4 GC does not zero freed ByteArray memory"`.

## 4. Property classes covered
| Class | (A) Std.Http | (B) stdhttp-verify | (C) LeanServer |
|---|---|---|---|
| Parsing | no (no proofs) | yes. Soundness and completeness of request-line, header, chunk-size, trailer and body parsers against RFC 9112 grammars, under stated `Std.Http` restrictions (spec/README.md table). | partial. Roundtrip lemmas for codecs (HPACK, Base64, QUIC VarInt) per `THREAT_MODEL.md:15-19`; not re-read individually here. |
| Functional correctness | no | yes for framing (`getSize_assigns`, `pulled_bodies`, `reads_requests`) | partial (state-mapping lemmas, size lemmas) |
| Safety invariant | no | yes: `nothing_after_rejection`, `no_desync`, `chunkBudget_run` over all operation sequences | partial: TLS FSM lemmas; the refinement shown is a state-label mapping |
| Liveness | no | partial: completeness results (`split_read`, `conn_read`, `conn_finish`) say requests within limits are read; `serve_split` gives equivalence of loop termination outcomes across splits | no |
| Refinement | no | partial: the patched loops equal the shipped behaviour by tests and bounded `native_decide` checks (`Spec/Real/Equiv.lean`, `URIEquiv.lean`); `serve_split` is over a hand-translated loop model | partial (`TLSRefinement.lean` maps labels) |
| Symbolic / computational security | no | no (request-smuggling resistance in the `no_desync` sense only) | no. Claimed via axioms (`ephemeral_key_independence`), which do not model an adversary. |
| Constant-time | no | no | no (`SideChannel.lean` provides best-effort zeroisation; the axiom admits a GC limitation) |

## 5. Link to implementation code
- (A) Lean is the implementation. There are no proofs about it upstream.
- (B) Lean is the implementation, in a modified copy. The proofs are about `fix/`, a copy of `Std.Http` from `nightly-2026-10-20` with these changes:
  - new total parsers in `Protocol/H1/ChunkLine.lean` and `RequestLine.lean`;
  - the 18 `partial def` read/write loops in `Protocol/H1.lean` made total with termination proofs;
  - URI parser helpers made total and public, with two behaviour fixes.
  The link back to shipped code works three ways:
  - Some theorems (`getSize_assigns`) are about unchanged upstream functions and are re-checked against the shipped toolchain via `./check-shipped.sh`.
  - Bounded exhaustive equivalence via `native_decide` (`Spec/Real/Equiv.lean`, `URIEquiv.lean`), e.g. "identical results on every target of length ≤ 6 over `/?a%1:@*.#`".
  - Differential testing (HTTP Garden against apache_httpd, go_stdlib, gunicorn, h2o, hyper and nginx; "150 seeds plus 4,987 mutated payloads (`harness/compare.py`) give the same results as the original server").
  The connection-loop result `serve_split` is about a model: "the model is a hand translation of `handle`, not derived from it" (spec/README.md, `LoopMain.lean` row).
- (C) Lean is the implementation, with an IO/FFI boundary. Sockets, epoll and signals are `@[extern]` C (`src/Network.c`). There is an optional OpenSSL crypto backend: `LeanServer/Crypto/FFI.lean:5-14` says the native functions "are semantically equivalent to the pure-Lean implementations ... Usage is controlled by `CryptoBackend` ... `crypto_backend=lean` for the verified pure-Lean fallback (default)". That equivalence is asserted, not proved. There is a `differential_crypto` target (`lakefile.toml:90`) and CBMC harnesses for the C code (`cbmc/`).

## 6. Trusted computing base and soundness hygiene
Static token counts, with `--` line comments stripped (block comments not stripped):
| Item | (A) src/Std/Http @ a2f1a64 | (B) spec/ | (B) fix/ | (C) LeanServer |
|---|---|---|---|---|
| Lean files / LOC | 47 / 13,726 | 60 / 33,247 | 51 / 15,113 | 108 / 38,581 |
| theorem/lemma decls | 25 | 1,840 | 37 | 929 |
| sorry | 0 | 0 | 0 | 2 tokens, both in comments/docs |
| admit | 0 | 0 | 0 | 5 tokens, all the identifier `admit` (connection pool), not the tactic |
| axiom | 0 | 0 | 0 | 5 real `axiom` declarations (listed in §3) |
| native_decide | 0 | 25 tokens (Bridge 1, Real/Equiv 11, Real/Fixed 2, Real/Original 6, Real/URIEquiv 5) | 0 | 416 tokens; about 379 tactic uses |
| implemented_by | 0 | 0 | 0 | 0 |
| extern | 0 | 2 tokens | 0 | 78 `@[extern` |
| opaque | 0 | 2 tokens | 1 | 78 `opaque` decls |
| partial | 28 tokens (21 `partial def`) | 4 tokens | 17 tokens (in `*Legacy` copies and `Data/Body/Stream.lean`; the H1 loops are total) | 5 `partial def` |

(B) labels the trust base per result: "kernel only" for most results and "compiler (`native_decide`)" for the bounded equivalence checks. The IPv6 case "takes libuv's answer as a hypothesis" because `IPv4Addr.ofString`/`IPv6Addr.ofString` are `@[extern]`.

(C) soundness observations from static reading, not machine-checked here:
1. `io_ref_atomic_ops : ∀ (descr : String), descr = "<fixed string>"` asserts that every string equals one fixed string. Instantiating it at two distinct literals gives their equality, which contradicts decidable string inequality. The axiom therefore makes the environment inconsistent (`False` is derivable). `io_ref_compound_not_atomic` and `gc_no_guaranteed_zeroization` have the same shape. All three are in modules imported by both root targets (`LeanServer.lean:66,85`, `LeanServerPure.lean:79,91`).
2. `ephemeral_key_independence` asserts that `scalarmult_base` is injective on 32-byte inputs. However `scalarmult` first applies `clamp` (`LeanServer/Crypto/X25519.lean:91-97`: `let b0 := bytes[0]! &&& 248`). Two keys that differ only in the low three bits of byte 0 therefore produce the same public key, so the axiom contradicts the code's own definitions.
3. `single_mutex_no_deadlock : (1 : Nat) * (1 - 1) = 0` is an arithmetic fact unrelated to locks.
4. `LeanServer/Proofs.lean:36` states "Total: 164 provas, 0 axiomas, 0 sorry, 0 vacuamente verdadeiros", and `THREAT_MODEL.md:11` states "0 `axiom`". Both conflict with the five axioms above.

## 7. Scale, effort, automation
- (B): 33,247 lines of spec/proof (1,840 theorems) about a 15,113-line patched implementation. The largest files include `Spec/Real/Machine.lean` (≥ 2,179 lines) and `Spec/Real/Absorb.lean` (≥ 2,775 lines). Effort is not reported. The work was AI-assisted (Claude), per the disclosures in §1. Automation: `simp`, `omega`, `decide`, `native_decide` for bounded checks. Testing: "every 2-way split position: 40 of 7,474 change the result", "3,000 random 2–5-way splits: 22 changes" (FINDINGS §1).
- (C): 38,581 Lean lines, 929 theorem/lemma declarations by grep. The README layer table lists Spec ~5,450 LOC. Effort is not reported.
- (A): 13,726 lines of implementation.

## 8. Limitations stated by the authors (verbatim)
- (B) spec/README.md, `Loop.lean` row: "Not covered: applications that respond before reading the whole body, other orders of application and network events, timeouts; the model is a hand translation of `handle`, not derived from it."
- (B) `Absorb.lean` row: "Not covered: pulls that hand out data ... and the buffer limit `maxBufferedInputBytes`, which by design applies to what a single read leaves buffered."
- (B) Next steps §3: "Not covered: absolute-form targets with schemes other than `http` and `https`, and what libuv's `inet_pton` accepts (the IPv6 case takes its answer as a hypothesis)."
- (B) `Real/Framing.lean` row: "`getSize` also rejects some requests the RFC would accept ... the theorems do not constrain rejections."
- (B) `Drain.lean` row: "Not proven: that the write loop treats `Lag` machines alike (it reads only the reader's state and message count, which they share; checked by reading the code)."
- (C) `THREAT_MODEL.md` §1: "The Lean 4 compiler translates Lean source to C code ... **Neither translation step is formally verified.**" §2: "The C FFI code (`src/Network.c`, `src/crypto_ffi.c`) is compiled by GCC/Clang. These compilers are not formally verified". `TLSSpec.lean:268`, on the DH axiom: "proving it would require formalising the arithmetic of Curve25519 over GF(2²⁵⁵ - 19), which is a separate project".

## 9. Independent evidence
- (B) produced upstream bug reports against (A):
  - lean4#15840, opened 2026-10-21 by eumemic. Its description, read in full on the issue page: "`Std.Http`'s HTTP/1.1 receiver rejects a valid chunked request with `400 Bad Request` when a read boundary falls inside a chunk extension (`5;a=b`) or a trailer field. The same bytes delivered in one read are accepted."
  - Fix PR #15841, shown as Open in the fetched (cached) page.
  - At lean4 master `a2f1a64` (2026-11-06), `optionalUnlessEof` is present in `src/Std/Http/Protocol/H1/Parser.lean:61`, which matches the PR's described change. How it was merged (PR #15841 or another commit) was not established from the shallow clone.
  - The README also lists #16035/#16080 (414 vs 400), #16021 and #16058/#16079 (URI). These were not opened here (see §15).
- (B) FINDINGS §1: "No split ever turned a rejection into acceptance or changed how an accepted request was read, so this is an availability/correctness bug, not a smuggling vector."
- (C): no third-party review, adoption or bug reports found. The soundness observations in §6 are this dossier's static reading.

## 10. Reproducibility
Not built: sandbox disk limit; queued for CI reproducibility job. A shared-disk ENOSPC occurred during toolchain installation for another dossier, and builds were then halted on instruction. Exact commands:
- (B) `cd stdhttp-verify/spec && lake build` (toolchain `nightly-2026-10-20`, depends on `../fix`), plus `./check-shipped.sh` for the shipped-`getSize` re-check.
- (C) `cd LeanServer && lake build LeanServerPure` (toolchain `v4.29.1`; full `lake build` also needs OpenSSL headers and the C objects in `lakefile.toml`).
- (A) is part of the Lean toolchain build and was not separately checked.
Recommended CI check for (C): `#print axioms` on `forward_secrecy_via_ephemeral_keys` and on an arbitrary theorem in a module downstream of `Concurrency.lean`, plus a two-line derivation of `False` from `io_ref_atomic_ops` to confirm observation §6.1.

## 11. Generalisation
- (B) shows a repeatable pattern for "check a shipped implementation against an RFC" in Lean when the implementation is itself Lean:
  - Find a bug with differential testing (HTTP Garden).
  - Make the implementation's `partial` loops total in a copy.
  - Prove grammar-level soundness/completeness, an invariant over all operation sequences (`reads_requests`), and a hyperproperty (segmentation independence, `serve_split`).
  - Tie the copy back to the shipped code with bounded equivalence (`native_decide`) and differential tests.
  Preconditions specific to this case: the target is pure Lean with a sans-I/O design, which makes "run over a list of operations" definable. Totalising `partial def`s required editing the code. The I/O loop needed a hand model. For a non-Lean codebase, the bounded-equivalence and differential-testing parts carry over; the in-kernel proofs about the real code do not, unless there is extraction or a semantics for that language.
- (C) is a "Lean is the implementation, axioms for the hard parts" pattern. Its documentation shows the risk of unchecked axioms (§6) and of theorem counts used as a proxy for assurance.
- (A) is an example of production Lean code whose `partial` definitions block direct proof, which is the precondition (B) had to remove.

## 12. Bearing on sub-claims (facts only, no verdict)
| Sub-claim | Fact from this reference | Source ID |
|---|---|---|
| H1 | RFC 9112 message grammar, framing (§6.3) and a receive state machine over arbitrary operation sequences and read splits are expressed in Lean (B). TLS handshake states are expressed in (C); cryptographic adversaries are not modelled in either. | S2, S3, S6 |
| H2 | (B) states kernel-checked proofs of `no_desync`, `nothing_after_rejection`, `pull_split`, `serve_split` (1,840 theorems, 0 sorry/axiom); not re-checked here. (C) "forward secrecy" is a restated axiom, and its axiom set appears inconsistent on static reading. | S2, S3, S6 |
| H3 | (B) proofs apply to a modified copy of shipped Lean library code. Some are re-checked against the shipped code; the rest are linked by bounded `native_decide` equivalence and differential tests. The connection loop is a hand-translated model. No mainstream-language implementation is involved. | S2, S3 |
| H4 | (B) depends on the implementation being Lean and sans-I/O. (C) depends on FFI equivalence that is asserted, not proved. | S2, S6 |
| H5 | (B) found and fixed a real implementation bug (lean4#15840) through differential testing first, then proved the fix. The segmentation bug is stated as "a counterexample to `pull_split` in the shipped server". | S2, S4, S1 |
| D | (B) "proven" = kernel-checked, with a per-result trust column ("kernel only" vs "compiler (`native_decide`)"). (C) uses "verified" for theorems that include axioms and arithmetic tautologies. | S3, S6 |

## 13. Conflicts
- (C) internal: "935 teoremas" (README:3), "914 verified theorems (0 `sorry`, 0 `axiom`)" (`THREAT_MODEL.md:11`, `LeanServer.lean` header comment), and "Total: 164 provas, 0 axiomas" (`LeanServer/Proofs.lean:36`), against 929 theorem/lemma declarations and 5 `axiom` declarations in the source.
- (C) README "sem spec-impl gap" against `FFI.lean`'s optional OpenSSL backend, which is asserted (not proved) to be "semantically equivalent", and against `THREAT_MODEL.md`, which lists the Lean compiler, C compiler and C FFI as unverified.
- (B) FINDINGS header says master `src/Std/Http` "has had no parser changes since 2026-07-24" (as of the nightly-2026-10-20 snapshot). Lean master at 2026-11-06 contains `optionalUnlessEof`, so (A) has changed since. The fetched PR page showed #15841 as Open, which may be stale cache.

## 14. Sources
| ID | Title | URL | P/S | Depth | Summary |
|---|---|---|---|---|---|
| S1 | leanprover/lean4 `src/Std/Http` @ a2f1a64, accessed 2026-11-07 | https://github.com/leanprover/lean4/tree/master/src/Std/Http | P | CODE | 47-file sans-I/O HTTP/1.1 server; 21 `partial def`s including the H1 read/write loops; 25 helper theorems; contains `optionalUnlessEof`. |
| S2 | eumemic/stdhttp-verify README and FINDINGS.md @ f19b903, accessed 2026-11-07 | https://github.com/eumemic/stdhttp-verify | P | FULL | Differential testing via HTTP Garden finds segmentation-dependent 400s plus three minor bugs; summary of proofs over a patched `Std.Http` copy; AI-assistance disclosure in `upstream/`. |
| S3 | stdhttp-verify `spec/README.md` and `spec/Spec/**` source @ f19b903, accessed 2026-11-07 | https://github.com/eumemic/stdhttp-verify/tree/main/spec | P | CODE | 33k lines, 1,840 theorems; per-result table with trust column; theorem statements quoted in §3. |
| S4 | lean4 issue #15840, accessed 2026-11-07 | https://github.com/leanprover/lean4/issues/15840 | P | FULL | Bug report with root cause (`optional (attempt …)` swallowing `.eof`) and a minimal repro. |
| S5 | lean4 PR #15841, accessed 2026-11-07 | https://github.com/leanprover/lean4/pull/15841 | P | FULL | Fix introducing `optionalUnlessEof`, one-field-per-call trailer parsing and regression tests; status shown as Open in the fetched page. |
| S6 | AfonsoBitoque/LeanServer source, README, THREAT_MODEL.md @ 24b916a, accessed 2026-11-07 | https://github.com/AfonsoBitoque/LeanServer | P | CODE | 38.6k-line Lean HTTPS server; TLS spec/refinement modules; five axioms including inconsistent `∀ descr, descr = "…"` forms; OpenSSL FFI backend. |

## 15. Unverified leads (excluded from evidence)
- lean4 issues #16035, #16021, #16058 and PRs #16079, #16080, listed in the stdhttp-verify README; not opened.
- Whether PR #15841 was merged as-is or superseded (the master code matches; merge record not read).
- LeanServer `docs/PROOF_GUIDE.md`, `cbmc/README.md` and `fuzz/`; not read.
