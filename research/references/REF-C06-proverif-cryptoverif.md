# REF-C06: ProVerif / CryptoVerif case studies (TLS 1.3, Signal, WireGuard, Noise Explorer), with the code-generation and implementation links

Kind: comparator (non-Lean). Access date for all sources: 2026-11-07. I only inspected the sources statically. ProVerif and CryptoVerif were not installed or run (see Section 10).

Source depth counts (Section 14): [FULL] 4 (S1, S3, S5, S6), [SECT] 2 (S7, S8), [CODE] 3 (S2r, S4r, S6r).

## 1. Identity
| Field | Value |
|---|---|
| Subject (protocol or algorithm) | Four bodies of work from the Inria Prosecco group and co-authors. (a) TLS 1.3 draft 18: ProVerif symbolic models (also covering TLS 1.2 composition), CryptoVerif computational proofs, and RefTLS, a JavaScript reference implementation whose core is translated to ProVerif [C06-S1, C06-S2r]. (b) Signal Protocol variant "SP" (close to TextSecure v3): ProVerif models compiled from ProScript (JavaScript) code deployed in Cryptocat, plus hand-written CryptoVerif models [C06-S3, C06-S4r]. (c) WireGuard (Noise IKpsk2) handshake plus transport data: CryptoVerif computational proof [C06-S5]. (d) Noise Explorer: compiles any Noise Handshake Pattern to a ProVerif model, and from the same parsed pattern to Go/Rust/Wasm implementations [C06-S6, C06-S6r]. |
| Spec it targets | (a) draft-ietf-tls-tls13-18. The repo also has draft-20 and RFC 8446 ProVerif models (`pv/tls13-draft20-only.pv`, `pv/tls13-rfc8446-only.pv`) [C06-S2r]. (b) Signal / TextSecure v3 as implemented in Cryptocat, with separate DH and signing keys [C06-S3]. (c) WireGuard whitepaper (NDSS 2017) / Noise IKpsk2 [C06-S5]. (d) Noise Protocol Framework, revision 33/34 [C06-S6, lines 1151-1167]. Repo README: "Version 1.0.7, based on Noise Protocol Revision 34" [C06-S6r]. |
| Organisation / authors | (a) Bhargavan, Blanchet, Kobeissi (Inria Paris), IEEE S&P 2017. (b) Kobeissi, Bhargavan, Blanchet (Inria Paris), IEEE EuroS&P 2017. (c) Lipp, Blanchet, Bhargavan (Inria Paris), IEEE EuroS&P 2019. (d) Kobeissi (Inria Paris / Symbolic Software), Nicolas (Symbolic Software), Bhargavan (Inria Paris), IACR ePrint 2018/766 (version dated April 17, 2019). |
| Primary URLs | https://bblanche.gitlabpages.inria.fr/publications/BhargavanBlanchetKobeissiSP2017.pdf ; https://bblanche.gitlabpages.inria.fr/publications/KobeissiBhargavanBlanchetEuroSP17.pdf ; https://bblanche.gitlabpages.inria.fr/publications/LippBlanchetBhargavanEuroSP19.pdf ; https://eprint.iacr.org/2018/766.pdf |
| Repo, commit SHA, last commit date, licence | (a) github.com/inria-prosecco/reftls @ a71a74538e8af80dec5f6719f9e726b8c654468a, 2026-06-02 ("Add ProVerif model and PVL for RFC8446 (#12)"). No top-level LICENSE file. (b) github.com/inria-prosecco/proscript-messaging @ a6e57dd0c5e6702b94b0c66fad5e155804878813, 2022-05-30. No top-level licence; `ps2pv/LICENSE` is an Inria BSD-style notice ("Copyright (c) 2012-2013 Institut National de Recherche en Informatique et Automatique"). (c) Models are distributed at https://cryptoverif.inria.fr/WireGuard and in the CryptoVerif distribution [C06-S5]. Not cloned; see Section 15. (d) github.com/symbolicsoft/noiseexplorer @ 76dec0fa6399b019451bf42c3d091a7958b78e17, 2026-04-07 ("Update errant version number"); GPL-3.0 (`LICENSE.md`). |
| Tool and version | ProVerif (applied pi-calculus, Horn-clause resolution; symbolic); CryptoVerif (game-based, computational). Versions stated: CryptoVerif 1.24 for the Signal replay countermeasure [C06-S3]; CryptoVerif 2.01 extensions for WireGuard [C06-S5]. The ProVerif version is not stated in (a)/(b)/(d). Translators: ProScript-to-ProVerif compiler `ps2pv` (OCaml) [C06-S4r]; Noise Explorer (Node.js, pattern parser plus templates `src/pv`, `src/go`, `src/rs`) [C06-S6r]. |
| Maturity | ProVerif and CryptoVerif are mature research tools. RefTLS README: "WARNING: Research Prototype: Do not use in production environments" [C06-S2r]. proscript-messaging README: "PS2PV, the ProScript-to-ProVerif compiler, is currently in a highly experimental implementation state. Bugs are numerous and to be expected." [C06-S4r]. Noise Explorer is maintained (commit 2026-04-07), and its README status table marks ProVerif, Go, Rust and Wasm output "Functional / Reliable / Tests" ✔ [C06-S6r]. AI provenance: `none-found`. Basis: no AGENTS.md, CLAUDE.md, .claude, .cursor or Copilot config in the reftls, proscript-messaging or noiseexplorer trees at the recorded SHAs, and no AI co-author trailers. These were depth-1 clones, so only the HEAD commit was checked. All four papers were peer-reviewed or preprinted in 2017-2019. |

## 2. Claim as stated by the authors (verbatim)
- TLS 1.3 [C06-S1, conclusion]: "the formal connections between our ProVerif models, CryptoVerif proofs, and JavaScript code are not as strong as could be desired... we are working on formal proofs of correctness for our translations." (I did not retain the abstract's exact wording from my notes, so it is not quoted here.)
- Signal / ProScript [C06-S3, Section 4]: "We currently do not formally prove translation soundness, so proofs of the resulting ProVerif model do not necessarily imply proof of the source code. Instead, we use model translation as a pragmatic tool..."
- proscript-messaging README [C06-S4r]: "The messaging protocol implementation included in this repository is exactly the same code that is used by the Cryptocat secure messaging software in production." and "The remaining directories can be used to generate actual ProVerif models from actual ProScript code".
- WireGuard [C06-S5, conclusion]: the analysis proves "correctness, secrecy, forward secrecy, mutual authentication, session uniqueness, channel binding, and resistance against replay, key compromise impersonation, and denial of service attacks." [wg.txt, lines ~1727-1728]
- Noise Explorer [C06-S6, lines 27-33]: "Noise Explorer can validate any Noise Handshake Pattern and then translate it into a model ready for automated verification and also into a production-ready software implementation written in Go or in Rust. We use Noise Explorer to analyze more than 57 handshake patterns. We confirm the stated security goals for 12 fundamental patterns and provide precise properties for the rest."

## 3. What is actually formalised

### 3.1 Common substrate
- ProVerif: protocols are applied pi-calculus processes. Messages are terms, and cryptography is constructors/destructors plus equations ("perfect black-boxes"). The Dolev-Yao attacker controls the public channel, and sessions are unbounded [C06-S6, lines 200-229]. Example DH equation used in Noise Explorer: `equation forall a:key, b:key; dhexp(b, dhexp(a, g)) = dhexp(a, dhexp(b, g)).` [C06-S6, lines 253-255]. ProVerif "may return that the security goal is true..., false with a counterexample, unable to conclude, or may fail to terminate" [C06-S1, Section 2].
- CryptoVerif: a sequence of games with explicit probability bounds under stated cryptographic assumptions. It is "semi-automated... requires human guidance" [C06-S1, Section 2].

### 3.2 TLS 1.3 (RefTLS) [C06-S1, C06-S2r]
- ProVerif threat model, beyond Dolev-Yao, includes weak DH groups, weak hashes (collisions), RSA decryption oracles, weak AE and leaky decryption, all modelled as events and compromise rules in `pv/tls-lib.pvl` (721 lines).
- Real query, `pv/tls13-draft18-only.pv:99-109` (application data authentication):
```
(* Main appData authentication query: every disjunct is required; commenting any of them results in "false" *)
(* Replay prevention relies on the recipient not accepting 2 messages with the same (n,ad);
   it is not explicitly proved in ProVerif. *)
query cr:random, sr:random, cr':random, sr':random, psk:preSharedKey, e:element,
      p:pubkey, n:bitstring, ad:bitstring, m:bitstring;
      event(ClientReceives(TLS13,cr,sr,psk,p,n,ad,m)) ==>
      event(ServerSends(TLS13,cr,sr,psk,p,n,ad,m)) ||
      (event(WeakOrCompromisedKey(p)) && (psk = NoPSK || event(CompromisedPreSharedKey(psk)))) ||
      event(ServerChoosesAE(cr,sr,p,TLS13,WeakAE)) ||
      event(ServerChoosesKEX(cr,sr,p,TLS13,DHE_13(WeakDH,e))) ||
      event(ServerChoosesHash(cr',sr',p,TLS13,WeakHash)).
```
- Real query, `pv/tls13-draft18-only.pv:85-91` (application data secrecy): `attacker(m_c(TLS13,cr,sr,p,psk)) ==> (event(WeakOrCompromisedKey(p)) && (psk = NoPSK || event(CompromisedPreSharedKey(psk)))) || event(ServerChoosesAE(cr,sr,p,TLS13,WeakAE)) || event(ServerChoosesKEX(cr,sr,p,TLS13,DHE_13(WeakDH,e))) || event(ServerChoosesHash(cr',sr',p,TLS13,WeakHash)).`
- README on reading the results: "the 'true' queries correspond to security goals like (Forward) Secrecy, Authenticity, Replay Prevention, and Unique Channel Identifiers... The 'false' queries correspond to queries that we expect to fail; they show that our verification is tight" [C06-S2r, README].
- CryptoVerif: key-schedule lemmas (`cv/KeySchedule1-3.cv`, `cv/HKDFexpand.cv`), initial handshake, PSK handshake and record protocol, composed manually by a hybrid argument [C06-S1, Section 6]. Real query, `cv/tls13-core-InitialHandshake.cv:277`: `query secret c_cats public_vars c_sats, c_ems, c_resumption_secret, ...`. Recorded results in `cv/results`: every file reports "All queries proved". The longest is `tls13-core-PSKandPSKDHE-NoCorruption` at 380.880 s [C06-S2r].
- CryptoVerif assumptions: GDH in prime-order groups; DH elements ≠ 0^len and ≠ label bitstrings (an issue fixed in draft 19); UF-CMA signatures; collision-resistant hash; HMAC as random oracle (extract) and PRF; AEAD IND-CPA + INT-CTXT [C06-S1, Section 6].
- CryptoVerif scope omits TLS 1.2, downgrade/negotiation, client certificates combined with PSK, and post-handshake authentication [C06-S1, Section 6].
- RefTLS code: `code/src/*.js`, 6,552 lines of Flow (`tls_formats.js` 1,909; `tls_crypto.js` 1,743; `tls_core_protocol.js` 932; `tls_handshake.js` 800; `tls_record.js` 498) [C06-S2r]. The paper splits this as about 3,000 trusted lines (mostly parsing), 2,500 untrusted and 1,000 verified core. The core is written in the ProScript subset (no loops or recursion; mutable state only through tables) and translated to an 800-line ProVerif model [C06-S1, Section 7]. The extracted 800-line model is not in the repo: `pv/` contains hand-written models only, and `code/` has no ProScript compiler [C06-S2r, static file listing].

### 3.3 Signal / Cryptocat (proscript-messaging) [C06-S3, C06-S4r]
- Pipeline (`Makefile`): `./ps2pv/jsdefp.native --pv ps/sp.js > secrecy-1-ab-oneway.pv` then `cat tl/secrecy-1-ab-oneway.pv >> secrecy-1-ab-oneway.pv`. The protocol processes come from code; the top-level process and queries come from hand-written files in `tl/` [C06-S4r].
- Sizes: `ps/sp.js` 648 lines; `ps2pv/*.ml` about 2,270 lines (`proverif.ml` 1,093; `typecheck.ml` 477); `tl/*.pv` 145-218 lines each; `cv/*.cv` 477-646 lines each [C06-S4r].
- Real ProVerif queries, `tl/authenticity-1-ab-oneway.pv:18-20`:
```
query a:key,b:key,m:bitstring; event(Recv(a, b, m)) ==> event(Send(a, b, m)).
query a:key,b:key,m:bitstring; event(Recv(a, b, m)).
query a:key,b:key,m:bitstring; event(Send(a, b, m)).
```
  The last two are reachability sanity checks. `tl/secrecy-1-ab-oneway.pv:16`: `query attacker(secMsg1).`
- Real CryptoVerif queries, `cv/TextSecure-AB-BA.m4.cv:195-208`:
```
query a:G,as:pkey,b:G,bs:pkey,sb:G,ob:G,a1:G,a2:G,m:bitstring;
      event inj:Recv(a,as,b,bs,sb,ob,a1,a2,m) ==> inj:Send(a,as,b,bs,sb,ob,a1,a2,m).
...
(* Without the anti-replay countermeasure, CryptoVerif should not
   prove the following query, because there is a replay attack. ... *)
query a:G,as:pkey,b:G,bs:pkey,sb:G,a1:G,a2:G,m:bitstring;
      event inj:Recv3(a,as,b,bs,sb,a1,a2,m) ==> inj:Send3(a,as,b,bs,sb,a1,a2,m).
query secret secb.
```
- Recorded results in `cv/results`: for `TextSecure-AB-BA.STD.NO_REPLAY_PROT`, "RESULT Could not prove event inj:Recv3(...) ==> inj:Send3(...)" (255.140 s, max rss 7.75 GB). The `REPLAY_PROT` variants report "All queries proved" (220-444 s; up to 14.7 GB RSS). The same pattern holds for the AA and KCI files [C06-S4r].
- CryptoVerif models one data message only: "Considering several data messages exceeds the current capabilities of CryptoVerif—the games become too big". They are hand-written, not generated from ProScript [C06-S3, Section 6].
- CryptoVerif assumptions: GDH / square GDH, Ed25519 UF-CMA, HKDF as random oracles, HMAC PRF, AES-GCM IND-CPA + INT-CTXT [C06-S3].

### 3.4 WireGuard (CryptoVerif) [C06-S5]
- New Curve25519 model with cofactor 8 and no public-key validation, where distinct public keys can yield equal shared secrets ("equivalent keys"). Indifferentiability lemmas for HKDF chains (Lemma 6 `hkdf_n`, Lemma 7 `chain_n`) are proved by hand "because CryptoVerif does not support loops".
- Modelling simplifications: timestamps and counters are supplied by the adversary ("CryptoVerif has no support for time"); mac1/mac2 are omitted from the main model, and the cookie is handled in a separate DoS model.
- Proved: correctness, mutual authentication (injective correspondences such as `inj-event(rcvd2(...)) ⇒ inj-event(sent2(...))`), KCI resistance from the key-confirmation message onward, replay protection, message secrecy and forward secrecy, session uniqueness, channel binding, DoS (cookie round trip, assuming mac is a PRF). Some psk dynamic-compromise cases use a manual argument "due to a limitation of CryptoVerif".
- Identity mis-binding holds only up to `pow_k` equivalence when all DH keys are compromised. The authors describe a corner-case MitM in which "A thinks it is connected to B′, B thinks it is connected to A′, but in fact they are both connected to each other... this vulnerability only appears in our precise model of Curve25519; it cannot be detected under a classic Diffie-Hellman assumption." [wg.txt, lines 1733-1745]. Recommendation: "we recommend that the static public keys be added as well" to the chaining key derivation [wg.txt, line 1750].
- Identity hiding fails with mac1 when the responder's public key is known. It holds without MACs if the AEAD hides associated data (IND$-CPA).
- Proof metrics: 36 instructions, 168 games, 14 min; the longest proofs take 16 and 18 min.

### 3.5 Noise Explorer [C06-S6, C06-S6r]
- Formal syntax (Fig. 3) and validity rules as typing rules (Fig. 4) for handshake patterns [C06-S6, lines 320-468]. Primitives and state functions come from fixed hand-written ProVerif headers. "Events, queries, protocol processes and the top-level process, however, are fully generated using translation rules" [C06-S6, lines 605-609]. Repo templates: `src/pv/0params.pv ... 8processes.pv`; Go templates `src/go/0params.go ... 6processes.go`; Rust templates `src/rs/*.rs` [C06-S6r].
- Security grades are expressed as event queries. Example from the paper (sender authentication, KCI resistance) [C06-S6, lines 857-859]: `RecvMsg(alice, bob, stage(d, sid), m) −→ (bob, c, stage(d, sid), m) ∨ LeakS(phase0, bob)`.
- Real generated query, `models/IKpsk2.noise.active.pv:390-400`:
```
query c:principal, m:bitstring, sid_a:sessionid, sid_b:sessionid, s:stage, b:bitstring, px:phasen, py:phasen, pz:phasen;
(* Message a: Authentication sanity *)
	event(RecvMsg(bob, alice, stagepack_a(sid_b), m)) ==> (event(SendMsg(alice, bob, stagepack_a(sid_a), m)));
(* Message a: Authentication 1 *)
	event(RecvMsg(bob, alice, stagepack_a(sid_b), m)) ==> (event(SendMsg(alice, c, stagepack_a(sid_a), m))) || ((event(LeakS(phase0, alice)))) || ((event(LeakS(phase0, bob))));
...
```
  The model is 646 lines, with one `query` block containing many queries.
- The repo ships 118 generated models (`models/*.noise.{active,passive}.pv`) and 118 ProVerif result files (`results/*.txt`). In `results/IKpsk2.noise.active.txt` there are 37 `RESULT` lines. Examples include "RESULT event(RecvMsg(bob,alice,stagepack_a(sid_b),m)) ==> event(SendMsg(alice,bob,stagepack_a(sid_a),m)) cannot be proved." (line 6462). This is consistent with message A of IKpsk2 having authentication grade 1 (Fig. 7). I did not reconcile each result line with the grade table [C06-S6r].
- Results: "all of the handshake patterns evaluated met the security goals postulated in the original Noise Specification. Verification times varied between less than 30 minutes for simpler... to more than 24 hours for some of the more ambitious patterns, such as IK." [C06-S6, lines 1171-1174]
- Session-ID approximation, stated limitation: generated queries use separate `sida`/`sidb` "due to performance limitations", which is "less accurate than the queries described previously" [C06-S6, lines 917-925].
- Forgery attack on the invalid pattern KXS (zero DH share plus AEAD nonce reuse) led to clarified validity-rule wording in Noise revision 34 [C06-S6, lines 1179-1284].

## 3a. Predicate semantics

### Tool-level meaning of the conclusion predicates
- ProVerif `query attacker(M)`, verbatim [C06-S7, Section 3.2.1]: "The tool allows the investigation of which terms are available to an attacker; and hence (syntactic) secrecy of terms can be evaluated with respect to a model." On output, "ProVerif returns 'RESULT not attacker(M) is true.' when M is secret, that is, the attacker cannot reconstruct M" [C06-S7, Section 4.3.1].
- ProVerif `event(e) ==> event(e')`, verbatim [C06-S7, Section 3.2.2]: "The query is satisfied if, for each occurrence of the event e(M1, . . . , Mj), there is a previous execution of e′(N1, . . . , Nk)." Variables before `==>` are universally quantified, and those only after it are existential. `inj-event`: "for each occurrence of the event e(M1, . . . , Mj), there is a distinct earlier occurrence of the event e′(N1, . . . , Nk)." Events "do not otherwise affect behavior". Their meaning comes entirely from where the modeller places them.
- ProVerif outcomes: "RESULT [Query] cannot be proved: This is a 'don't know' answer" [C06-S7, Section 3.3.1]. So "cannot be proved" lines in result files are not proofs of falsity unless an attack trace is reconstructed.
- CryptoVerif `secret x`, verbatim [C06-S8]: "show that the array x is indistinguishable from an array of independent random numbers (by several test queries), even when the variables in l are public". Correspondence: "CryptoVerif shows that, for all values of x1, . . . , xj, if M is true then M′ is true, even when the variables in l are public". These hold computationally, up to the probability bound CryptoVerif outputs, under the assumptions declared in the model (Section 3.2-3.4).

### TLS 1.3 headline: appData authentication (`pv/tls13-draft18-only.pv:102-109`)
- Conclusion predicate: `event(ServerSends(TLS13,cr,sr,psk,p,n,ad,m))`, or one of the weak/compromise events. `ServerSends` is declared at `pv/tls-lib.pvl:666` and emitted at `:693` and `:713`, immediately before `out(io,aead_enc(a,ks,n,ad,mesg))`.
- Strongest premise: `event(ClientReceives(TLS13,cr,sr,psk,p,n,ad,m))`, emitted at `pv/tls-lib.pvl:720` after `let f = aead_dec(a,ks,n,ad,c)` succeeds. The disjuncts cover:
  - `WeakOrCompromisedKey(p)` (emitted at `:157`, `:163`, `:191-192` when the attacker supplies or obtains a long-term key) combined with `psk = NoPSK || CompromisedPreSharedKey(psk)`;
  - the server choosing a weak AE, a weak DH group or a weak hash.
- Guaranteed: in the symbolic model, a TLS 1.3 client that accepts application data m under the session parameters received it from a server that sent m with matching parameters, unless one of the listed weaknesses or compromises occurred.
- Not guaranteed: replay protection ("it is not explicitly proved in ProVerif", `:100-101`), parsing, computational security, and any property of RefTLS code beyond what the unverified ProScript→ProVerif translation preserves (Sections 5-6).

### TLS 1.3 CryptoVerif headline: `query secret c_cats public_vars ...` (`cv/tls13-core-InitialHandshake.cv:277`)
- Conclusion: the client application traffic secret is indistinguishable from random, with the listed variables public (C06-S8 semantics).
- Premises: GDH, UF-CMA, collision resistance, ROM/PRF for HKDF and HMAC, AEAD IND-CPA+INT-CTXT (Section 3.2). Composition across the three pieces is a manual hybrid argument [C06-S1].

### Signal headline: authenticity and secrecy (`tl/authenticity-1-ab-oneway.pv:18`, `tl/secrecy-1-ab-oneway.pv:16`)
- Conclusion predicates: `event(Send(a, b, m))` and `not attacker(secMsg1)`. Both events are declared and placed by hand in the `tl/` top-level file, not in the code:
  - `Send` (`tl/authenticity-1-ab-oneway.pv:57-60`) fires after the generated `fun_send(initiatorIdentityKey, them, secMsg1)` returns a valid output (`:53-55`);
  - `Recv` (`:99-103`) fires after the generated `fun_recv(...)` returns `Object_msg_get_valid(o2) = true` (`:96-98`).
- Guaranteed: in the symbolic model of the translated `ps/sp.js` functions, a message accepted by `fun_recv` was produced by `fun_send` with matching identity keys, and `secMsg1` is not derivable by the attacker.
- Not guaranteed: correspondence between `fun_send`/`fun_recv` in the model and the JavaScript actually executed. Translation soundness is not proved [C06-S3]. Events and the top-level process are hand-written. The non-injective query does not exclude replay; replay is addressed by the injective CryptoVerif query (`cv/TextSecure-AB-BA.m4.cv:204-205`).

### WireGuard headline: mutual authentication (injective correspondences) [C06-S5]
- Conclusion: `inj-event(sent2(...))` given `inj-event(rcvd2(...))` and the analogous pairs. The CryptoVerif files were not inspected (Section 15), so I give no file:line. The meaning of the guarantee follows C06-S8, under the Curve25519 model with equivalent keys and adversary-supplied timestamps.

### Noise Explorer headline: per-message authentication grades (`models/IKpsk2.noise.active.pv:390-406`)
- Conclusion predicates: `event(SendMsg(alice, c, stagepack_a(sid_a), m))` or `LeakS(...)` disjuncts. The events are declared at `:381-383`. `SendMsg` is emitted in the generated initiator process before sending message a (`:502`), and `RecvMsg` after reading (`:509`, `:524`, `:556`, `:570`). `LeakS(phase0/phase1, me)` is emitted when the static key is revealed (`:535`, `:539`).
- Guaranteed: grades as defined in [C06-S6, Section 4], in the generated symbolic model. Separate session IDs `sid_a`/`sid_b` make the queries weaker than the paper's single-`sid` definitions [C06-S6, lines 917-925].
- Not guaranteed: anything about the generated Go/Rust code [C06-S6, lines 1325-1327].

## 4. Property classes covered
| Class | Status | Evidence |
|---|---|---|
| Functional correctness | partial | WireGuard: "correctness" proved in a separate sequential model [C06-S5]. RefTLS interop tests with NSS/BoringSSL (testing, not proof) [C06-S1]. Noise generated code is tested against Cacophony vectors (testing) [C06-S6, lines 1297-1299]. |
| Safety (trace properties) | proved (in the models) | Correspondence and authentication queries in all four (Section 3). |
| Liveness | no | Not addressed in any source read. |
| Refinement (model ↔ code) | assumed | ProScript→ProVerif and Noise pattern→{ProVerif, Go, Rust} are translations, with soundness explicitly not proved [C06-S3; C06-S6, lines 1325-1327]. |
| Symbolic security | proved (in the models) | ProVerif in (a), (b), (d). |
| Computational security | proved (in the models, under stated assumptions, with manual steps) | CryptoVerif in (a), (b), (c), with explicit assumptions (Section 3). |
| Parsing / serialisation | assumed | "our analysis assumes that message serialization and parsing is correct; it won't find any attacks that rely on parsing ambiguities or bugs" [C06-S1, Section 4]. RefTLS parsing code is trusted [C06-S1, Fig. 6]. Noise models use abstract `concat3`/`deconcat3` [C06-S6, lines 286-288, 662-677]. |
| Constant-time / side channels | tested (Noise generated code only) | Noise generated code is "tested to perform fully in constant time" (testing claim) [C06-S6, lines 1307-1308]. Go uses `crypto/subtle.ConstantTimeCompare` (`implementations/go/IKpsk2.noise.go:104`). Rust links HACL* via the `hacl-star = "0.1.0"` crate (`implementations/rs/IKpsk2/Cargo.toml:10`) [C06-S6r]. |

## 5. Link to implementation code
- WireGuard (CryptoVerif): none. The paper analyses the protocol as specified in the whitepaper, and I found no implementation link in the full text [C06-S5].
- TLS 1.3 RefTLS: the verified core (about 1,000 lines of ProScript in Flow) is compiled by an unverified ProScript-to-ProVerif compiler into an 800-line model, which is verified in "several hours". Parsing (about 3,000 lines), the crypto library and the runtime are trusted. RefTLS interoperates with NSS and BoringSSL (draft 18), was integrated into the Brave browser, and is about 2× slower than OpenSSL [C06-S1, Section 7]. The CryptoVerif models are hand-written and not linked to the code [C06-S1].
- Signal / Cryptocat: `ps/sp.js` (648 lines) is the deployed Cryptocat protocol code according to the README. `ps2pv` compiles it to ProVerif, and the queries and top-level process are appended by hand [C06-S4r]. "We currently do not formally prove translation soundness" [C06-S3]. A previous attempt via DJS2PV produced a 3,800-line model "that could not verify" [C06-S3].
- Noise Explorer: one parser and pipeline emits both the ProVerif model and the Go/Rust/Wasm code. Verbatim: "while we are able to link the protocol logic of the generated implementations to that of the generated symbolic models, we do not prove anything with regards to the functional correctness or security of the implementation code itself. That is left for future work, and likely can be best accomplished by using specialized frameworks for writing formally verifiable software, such as F⋆" [C06-S6, lines 1325-1328]. Generated IKpsk2 sizes: Go 341 lines and Rust 790 lines (paper Fig. 8) [C06-S6, lines 1240-1259]. Current repo: `implementations/go/IKpsk2.noise.go` has 537 lines [C06-S6r].
- Direction of link: in (a) and (b), code → model (extraction-like translation). In (d), pattern → model and pattern → code (co-generation from a common source; no code-to-model relation is checked).

## 6. Trusted computing base and soundness hygiene
- Proof tools: the ProVerif and CryptoVerif implementations are trusted (OCaml; no proof certificate checked by a small kernel in these works). None of the sources says otherwise.
- Translators: the ProScript-to-ProVerif compiler (unverified; "highly experimental... Bugs are numerous" [C06-S4r]) and the Noise Explorer generator (unverified) are both in the TCB.
- RefTLS verbatim: "we did not verify our message parsing code or cryptographic libraries, and our verification results rely on the correctness of the unverified ProScript-to-ProVerif compiler" [C06-S1, Section 7].
- Signal verbatim: "we have not formally verified the soundness of the cryptographic primitives themselves... not formally verified the Electron framework's isolation code... V8... Node.js"; "they do not imply the absence of security bugs, since we still have a large trusted computing base" [C06-S3, Section 7/8].
- Manual proof steps outside the tool: TLS 1.3 composes CryptoVerif pieces by a manual hybrid argument [C06-S1]. WireGuard proves HKDF-chain lemmas manually and covers some psk-compromise cases manually [C06-S5].
- Hand-written query and top-level files sit outside the code-to-model translation: `tl/*.pv` in proscript-messaging [C06-S4r] and hand-written headers `src/pv/0-7*.pv` in Noise Explorer [C06-S6, lines 605-607].
- Lean-style counters (sorry/axiom/native_decide) do not apply. The equivalent concerns are stated assumptions and manual steps (above) and "cannot be proved" results, which are expected for some sanity or negative queries [C06-S2r README; C06-S6r results].

## 7. Scale, effort, automation
| Work | Model size | Run time | Human effort | Automation |
|---|---|---|---|---|
| TLS 1.3 ProVerif [C06-S1] | 1,030 lines (library 400, TLS 1.2 200, TLS 1.3 250, queries 50+180). Repo: `tls-lib.pvl` 721 lines; per-variant `.pv` 133-145 lines | "All proofs complete in about 70 minutes"; the composite model "takes hours on a powerful workstation" | "about 3 person-weeks for a ProVerif expert" | Push-button after modelling |
| TLS 1.3 CryptoVerif [C06-S1] | 1,895 lines (key schedule 570, initial handshake 550, PSK 625, record 150) | "about 6 minutes" (paper). `cv/results` lists 0.02-380.9 s per file [C06-S2r] | "about 5 person-weeks for a CryptoVerif expert" | Semi-automated, with proof hints |
| RefTLS [C06-S1] | 6,552 lines of Flow (repo count); extracted model 800 lines | "several hours" | not stated | Compiler + ProVerif |
| Signal ProVerif [C06-S3, C06-S4r] | sp.js 648 lines + `tl` 145-218 lines | 3m30s to 26h10m08s (README benchmarks; e.g. `authenticity-1-abm-twoway` 26h 10m 08s) | not stated | Compiler + ProVerif |
| Signal CryptoVerif [C06-S3, C06-S4r] | 477-646 lines per file | 3-7 min per proof (paper); 194-444 s (recorded results) | not stated | Manual guidance |
| WireGuard CryptoVerif [C06-S5] | not given as a line count | 14 min (36 instructions, 168 games); longest 16 and 18 min | not stated | Semi-automated + manual lemmas |
| Noise Explorer [C06-S6] | IKpsk2 model 646 lines (generated) | <30 min (NN) to >24 h (IK) | Generation is automatic; headers hand-written once | Fully automatic per pattern |

## 8. Limitations stated by the authors (verbatim)
- TLS 1.3: "our analysis assumes that message serialization and parsing is correct; it won't find any attacks that rely on parsing ambiguities or bugs... the same approach is taken by Tamarin [35]. In contrast, miTLS [21] includes a fully verified parser" [C06-S1].
- TLS 1.3 CryptoVerif: "unable to prove secrecy of the keys when psk is compromised after the end of the session... due to the current limitations of CryptoVerif" [C06-S1]. It also cannot prove authentication of the early traffic secret for the client, or replay protection for 0-RTT [C06-S1].
- TLS 1.3 conclusion: "the formal connections between our ProVerif models, CryptoVerif proofs, and JavaScript code are not as strong as could be desired" [C06-S1].
- Signal: "Considering several data messages exceeds the current capabilities of CryptoVerif—the games become too big" [C06-S3]. Proving ProScript→ProVerif soundness "remains future work" [C06-S3].
- WireGuard: "CryptoVerif has no support for time"; Lemma 7 is proved by hand "because CryptoVerif does not support loops" [C06-S5].
- Noise Explorer: "we do not prove anything with regards to the functional correctness or security of the implementation code itself" [C06-S6, lines 1326-1327]. "The full Noise specification extends somewhat beyond the description given as part of this work, including features such as 'identity hiding' and 'dummy keys.'" [C06-S6, lines 781-784]. The session-ID approximation is "less accurate" [C06-S6, lines 917-921].
- RefTLS README: "Replay prevention relies on the recipient not accepting 2 messages with the same (n,ad); it is not explicitly proved in ProVerif." (`pv/tls13-draft18-only.pv:100-101`) [C06-S2r].

## 9. Independent evidence
- Attacks and design changes adopted by standards bodies:
  - TLS 1.3: the findings fed into draft 13 (resumption context; removal of 0-RTT client-certificate authentication) and draft 19 (DH element check) [C06-S1].
  - Noise: revision 34 adopted Noise Explorer's results and the clarified validity rule [C06-S6, lines 1162-1176, 1275-1284].
  - Signal: a novel KCI with a compromised signed pre-key, and a replay when the one-time pre-key is absent. Separately, Telegram and Windows Phone issues were reported via a bug bounty [C06-S3].
- Cross-tool comparison by the WireGuard authors:
  - The Donenfeld-Milner Tamarin analysis "concludes the absence of identity mis-binding attacks even if all keys are compromised, because their model does not include equivalent public keys. We disprove this property by considering a precise model of Curve25519." [wg.txt, lines ~1790-1796]
  - Dowling-Paterson's manual proof (PRF-ODH) also "does not find the identity mis-binding issue" [C06-S5, Related Work].
- Girol's Tamarin analysis of Noise "confirmed some of Noise Explorer's findings" [C06-S6, lines 179-181]. I did not read it (Section 15).
- Deployment (not verification evidence): RefTLS in Brave; ProScript Signal code in Cryptocat [C06-S1, C06-S4r].

## 10. Reproducibility
- Status: not run: sandbox disk limit. I installed and ran no tools. The repos were shallow-cloned (`--depth 1 --filter=blob:limit=1m`), inspected statically, then deleted.
- Exact commands per the repos:
  - TLS ProVerif: `proverif -lib tls-lib <filename>` and `proverif -lib tls-lib <filename> | tee results.txt | grep ^RESULT` (reftls README; "verifying this model takes a long time and a significant amount of RAM even on powerful workstations").
  - TLS CryptoVerif: `cryptoverif -lib tls-lib <filename>`. The README notes that "Up-to-date TLS models are included in the CryptoVerif distribution, in the subdirectory `examples/tls13/`".
  - Signal: `opam install ocamlbuild merlin menhir ulex pcre; cd ps2pv; make; cd ..; make <property>-<messages>-<principals>-<direction>` then `proverif <generated>.pv`. CryptoVerif uses `cv/runcv`.
  - Noise Explorer: `cd src; make dependencies; make parser; make models` then `proverif models/IKpsk2.noise.active.pv`. Code generation: `node noiseExplorer --generate=go --pattern=<file>`.
- Expected resources from recorded outputs: CryptoVerif Signal runs used up to about 14.7 GB RSS [C06-S4r cv/results]. One Signal ProVerif query took 26 h [C06-S4r README]. Some Noise patterns took more than 24 h [C06-S6].

## 11. Generalisation
- Noise Explorer is the only one of the four that generalises by construction. It accepts any valid Noise pattern, but only within Noise's restricted language (DH, AEAD, hash; "Noise does not currently support any signing operations") [C06-S6, line 308].
- ProScript→ProVerif works only for code written in the ProScript subset (no loops or recursion, no extensible objects, state-passing style). Both RefTLS and Cryptocat were written in that subset on purpose [C06-S1, C06-S3]. There is no evidence in these sources that it applies to existing, non-ProScript codebases.
- CryptoVerif models are hand-built per protocol, and the authors report capacity limits: one data message only for Signal, no loops, no time [C06-S3, C06-S5].
- None of the four sources verifies parsing, crypto primitives or the runtime. HACL* is used as a verified external primitive in the Noise Rust output [C06-S6, lines 1309-1311].

## 12. Bearing on sub-claims (facts only, no verdict)
| Sub-claim | Fact | Source ID |
|---|---|---|
| H1 expressiveness | ProVerif expressed TLS 1.3 + 1.2 composition with weak-crypto threat models; CryptoVerif expressed computational secrecy and authentication with explicit assumptions. Noise patterns translate to full applied-pi models automatically. | C06-S1, C06-S6 |
| H1 expressiveness | Expressiveness limits stated: CryptoVerif has no loops, no time, and cannot handle more than one Signal data message. Noise Explorer does not model identity hiding or dummy keys. | C06-S3, C06-S5, C06-S6 |
| H2 provability | Recorded CryptoVerif outputs report "All queries proved" for the TLS files and the Signal REPLAY_PROT variants. Without the countermeasure, "Could not prove" the injective Recv3/Send3 query, matching the replay attack. | C06-S2r, C06-S4r |
| H2 provability | ProVerif runs range from minutes to more than 24 h (Noise IK) and 26 h (Signal ABM two-way). Some CryptoVerif secrecy goals were unprovable "due to the current limitations of CryptoVerif". | C06-S1, C06-S4r, C06-S6 |
| H3 implementation link | RefTLS and Cryptocat: code → ProVerif via an unverified compiler, with parsing, crypto and runtime trusted. Noise: co-generated code with no proof about the code. WireGuard: no code link. | C06-S1, C06-S3, C06-S5, C06-S6 |
| H4 generality | Translation applies only to code in the ProScript subset, or to protocols expressible as Noise patterns. | C06-S1, C06-S3, C06-S6 |
| H5 inference H1/H2⇒H3 | The authors state that model proofs "do not necessarily imply proof of the source code" and "we do not prove anything with regards to the functional correctness or security of the implementation code itself". | C06-S3, C06-S6 |
| D definitions | "Verified" here means a ProVerif query proved true in the symbolic model, or a CryptoVerif game sequence with a probability bound under stated assumptions. RefTLS calls its core "verified" while its parser and compiler are trusted. | C06-S1, C06-S2r |
| D definitions | Noise Explorer README marks generated Go/Rust as "Reliable"/"Tests" ✔. The paper calls them "production-ready" and "safe to use" without proofs about the code. | C06-S6, C06-S6r |

## 13. Conflicts (recorded, not resolved)
- WireGuard identity mis-binding: the Donenfeld-Milner Tamarin analysis concluded there is no mis-binding even with all keys compromised. Lipp et al. "disprove this property" under a precise Curve25519 model [C06-S5]. Cross-dossier link: REF-C05 covers Tamarin.
- TLS 1.3 post-handshake authentication: Cremers et al. (REF-C05, C05-S1) state that the ProVerif draft-18 analysis did not uncover the client-authentication mismatch. Bhargavan et al. state that they "did not consider" post-handshake authentication and that the Tamarin work did not consider 0-RTT client-certificate authentication or 0.5-RTT data [C06-S1]. These are scope differences reported by each side.
- RefTLS "verified" vs. extracted model: the paper reports an 800-line ProVerif model extracted from the code [C06-S1]. The public repo contains only hand-written ProVerif models and no extracted model or compiler [C06-S2r]. The extracted artefact may live elsewhere (proscript-messaging's `ps2pv`); I did not find it in reftls.
- Noise generated-code sizes: the paper reports IKpsk2 Go at 341 lines [C06-S6, Fig. 8]. The current repo file has 537 lines [C06-S6r]. This is likely a version difference and is unresolved.
- Noise Explorer pattern count: the abstract says "more than 57 handshake patterns" [C06-S6]. The repo ships 59 pattern pairs (118 models) [C06-S6r].

## 14. Sources
| ID | Title | URL | P/S | Depth | Summary |
|---|---|---|---|---|---|
| C06-S1 | Bhargavan, Blanchet, Kobeissi, "Verified Models and Reference Implementations for the TLS 1.3 Standard Candidate", IEEE S&P 2017 | https://bblanche.gitlabpages.inria.fr/publications/BhargavanBlanchetKobeissiSP2017.pdf | P | [FULL] | ProVerif models of TLS 1.3 drafts and TLS 1.2 composition with rich weak-crypto threat models, and CryptoVerif proofs of draft 18. Also RefTLS, a JS implementation whose ProScript core is compiled to ProVerif. Parsing, crypto and the compiler are trusted. |
| C06-S2r | inria-prosecco/reftls repository @ a71a7453 | https://github.com/inria-prosecco/reftls | P | [CODE] | Hand-written ProVerif models (draft 18, draft 20, RFC 8446, TLS 1.2) and CryptoVerif files with recorded "All queries proved" results. Also 6,552 lines of Flow source. No licence; no extracted model. |
| C06-S3 | Kobeissi, Bhargavan, Blanchet, "Automated Verification for Secure Messaging Protocols and their Implementations: A Symbolic and Computational Approach", IEEE EuroS&P 2017 | https://bblanche.gitlabpages.inria.fr/publications/KobeissiBhargavanBlanchetEuroSP17.pdf | P | [FULL] | ProScript subset of JS, compiled to ProVerif, applied to Signal-variant code used in Cryptocat. Finds novel KCI and replay issues. Hand-written CryptoVerif models for one message. Translation soundness is not proved. |
| C06-S4r | inria-prosecco/proscript-messaging repository @ a6e57dd0 | https://github.com/inria-prosecco/proscript-messaging | P | [CODE] | `ps/sp.js` (Cryptocat code), the OCaml `ps2pv` compiler, hand-written `tl/` query files, and CryptoVerif models with recorded results. These show the replay query unprovable without the countermeasure. |
| C06-S5 | Lipp, Blanchet, Bhargavan, "A Mechanised Cryptographic Proof of the WireGuard Virtual Private Network Protocol", IEEE EuroS&P 2019 | https://bblanche.gitlabpages.inria.fr/publications/LippBlanchetBhargavanEuroSP19.pdf | P | [FULL] | CryptoVerif proof of WireGuard/IKpsk2 with a precise Curve25519 model. Finds an identity mis-binding corner case missed by the Tamarin and manual analyses. No implementation link. |
| C06-S6 | Kobeissi, Nicolas, Bhargavan, "Noise Explorer: Fully Automated Modeling and Verification for Arbitrary Noise Protocols", IACR ePrint 2018/766 (Apr 2019) | https://eprint.iacr.org/2018/766.pdf | P | [FULL] | Formal syntax and validity rules for Noise, plus automatic ProVerif model generation for 57+ patterns. Co-generates Go/Rust code; the authors state that nothing is proved about that code. |
| C06-S6r | symbolicsoft/noiseexplorer repository @ 76dec0fa | https://github.com/symbolicsoft/noiseexplorer | P | [CODE] | GPL-3.0 tool with 118 generated ProVerif models and result files, generated Go/Rust/Wasm implementations, and the templates `src/pv`, `src/go`, `src/rs`. The Rust output links the hacl-star crate. |

| C06-S7 | ProVerif 2.05 manual, "ProVerif: Cryptographic protocol verifier in the formal model" (Sections 3.2.1 Reachability and secrecy; 3.2.2 Correspondence assertions; 3.3.1 result interpretation; 4.3.1 secrecy queries) | https://bblanche.gitlabpages.inria.fr/proverif/manual.pdf | P | [SECT] | Defines `query attacker(M)` as derivability of M by the attacker, and `event(e) ==> event(e')` as "if e has been executed, then e' has been previously executed", with inj-event requiring a distinct earlier event per execution. Also defines the three outcomes true/false/cannot be proved. |
| C06-S8 | CryptoVerif User Manual (Blanchet, Cadé, Fromherz, Jacomme; version not stated on the title page), section listing the available queries | https://bblanche.gitlabpages.inria.fr/CryptoVerif/manual.pdf | P | [SECT] | Defines `secret x` as indistinguishability of the array x from independent random numbers (`onesession`: a single test query). For correspondences `M ==> M'`, "CryptoVerif shows that, for all values of x1, . . . , xj, if M is true then M' is true"; `inj-event` gives injective correspondences. |

[FULL]: 4 (S1, S3, S5, S6). [SECT]: 2 (S7, S8). [CODE]: 3 (S2r, S4r, S6r).

## 15. Unverified leads (excluded from evidence)
- Cadé & Blanchet, generating OCaml implementations from CryptoVerif models (verified SSH), JCS 2015, cited in C06-S1 related work. Not read.
- cv2fstar (CryptoVerif → F*) and Benjamin Lipp's PhD thesis. Not read.
- Inria research report RR-9040 (full version of C06-S1) and the WireGuard long version (C06-S5 ref [15]). Not read.
- Dowling & Paterson, "A cryptographic analysis of the WireGuard protocol", ePrint 2018/080. Not read.
- Girol, ETH MSc thesis 2019, Tamarin analysis of Noise. Not read.
- Donenfeld & Milner, Tamarin WireGuard models (https://www.wireguard.com/formal-verification/). Not read.
- CryptoVerif distribution `examples/tls13/` and https://cryptoverif.inria.fr/WireGuard model files. Not downloaded (disk limit).
- FS2PV (F# → ProVerif), cited in C06-S1. Not read.
