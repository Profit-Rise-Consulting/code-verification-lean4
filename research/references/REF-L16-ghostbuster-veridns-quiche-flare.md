# REF-L16: GhostBuster (BGP), VeriDNS (DNS), Lean Squad quiche (QUIC), flare/formal (Mojo networking)

Source depth counts (section 14): 0 [FULL], 2 [SECT], 4 [CODE]. Evidence level: static-inspected (not built). AI provenance: author-reported (VeriDNS, quiche, flare); none-found for GhostBuster.

This dossier covers four separate Lean 4 networking artefacts. The source research files grouped them together. They are identified as follows:
- (G) **GhostBuster**, nsg-ethz/ghostbuster. A SIGCOMM 2026 runtime verifier for BGP routers, written in Rust, with a 309-line Lean proof of the algorithm's correctness argument.
- (V) **VeriDNS**, BasisResearch/VeriDNS. A recursive DNS resolver written in Lean 4 and proved against RFC-derived specifications; the README says it was written "by Claude". Name collision: a different "VeriDNS" (Du, Hu, Wang, Guo, *Computer Networks* 275:111929, 2026, "incremental distributed verification of DNS configurations") is unrelated and is not Lean work (§15).
- (Q) **quiche Lean work**, dsyme/quiche `formal-verification/`. "Lean Squad", an automated agent pipeline, writes Lean functional models of Rust functions from Cloudflare's quiche (QUIC + HTTP/3) and proves theorems about them.
- (F) **flare**, ehsanmok/flare `formal/`. Lean 4 models transliterated from a Mojo networking library (HTTP/1.1/2/3, QUIC, WebSocket), with RFC specs, proofs, source-hash anchors and differential tests.

## 1. Identity
| Field | Value |
|---|---|
| Subject (protocol or algorithm) | (G) BGP router compliance: whether observed outgoing BGP messages can be explained by some reordering of incoming ones. (V) Recursive DNS resolution over UDP (TCP code is also present; see §13): wire codec, cache, resolver state machine, IO loop. (Q) quiche QUIC/HTTP/3 algorithms: varints, octets, packet-number decode, congestion control (NewReno, CUBIC, BBR2), stream buffers, H3 frames, QPACK integers. (F) Mojo networking stack: codecs, socket/reactor machines, HTTP/1.1, HTTP/2 + HPACK, QUIC, QPACK, HTTP/3, WebSocket, application-layer connection machine, concurrency. |
| Spec it targets (e.g. RFC 8446 section) | (G) the paper's own "witness" semantics of a BGP router (`spec : State → Msg`), not an RFC text. (V) RFC 1034, 1035, 2181 §5.4.1, 2308, 5452 §9.1, with RFC text embedded and checked via `include_rfc`. (Q) RFC 9000 (e.g. §A.3 packet-number decoding, §2.1 stream IDs), RFC 9114, RFC 9204, RFC 9002. (F) RFC 9000, 9002, 9113 §5.1, 9114, 9204, 7541 App. B, plus flare's own docs. |
| Organisation / authors | (G) Pietro Ronchetti, Tibor Schneider, Laurent Vanbever (ETH Zürich NSG); Lean file committed by `Pietroro <pietroro@student.ethz.ch>`. (V) Basis Research; last commit by Kiran Gopinathan. (Q) Don Syme's fork; the paper's author line is "Lean Squad / Automated Formal Verification Agent". (F) Ehsan M. Kermani (`ehsanmok`). |
| Primary URLs | (G) https://github.com/nsg-ethz/ghostbuster ; paper page https://nsg.ee.ethz.ch/publications/2026-05-13-when-static-verification-is-not-enough-revealing-bgp-bugs-at-runtime-20-500-11850-800202/ . (V) https://github.com/BasisResearch/VeriDNS ; docs https://basisresearch.github.io/VeriDNS/ . (Q) https://github.com/dsyme/quiche/tree/main/formal-verification (fork of cloudflare/quiche). (F) https://github.com/ehsanmok/flare/tree/main/formal |
| Repo, commit SHA, last commit date, licence | (G) `7564d6e4e5e9a25777c0d5f2194ae1a3996ebcd3`, 2026-07-30, GPL-3.0. (V) `74bb5f3f1454a8de2744e69f8af18fe48e0e9f2e`, 2026-07-17, no LICENSE file at root. (Q) `5a8195ebf9bf96383c3e6dbfe92105e93dbc637b`, 2026-06-03, no LICENSE file in the sparse checkout of `formal-verification/` (upstream quiche is BSD-2-Clause; not checked here). (F) `576799df50c8c525db4d558e29454365e885b213`, 2026-11-02, MIT. |
| Lean toolchain (or other tool and version) | (G) `v4.32.0-rc1` + Mathlib. (V) `v4.31.0-rc2` + Verso, MD4Lean, illuminate, batteries, plausible. (Q) `v4.29.0`, no Mathlib (REPORT says 4.29.1). (F) `v4.33.0`, no Mathlib. |
| Maturity | (G) Research artefact for a peer-reviewed paper. (V) Research prototype. README says "written and verified in Lean 4 by Claude!", and the repo has a `CLAUDE.md`. (Q) Research. "🔬 *Written by Lean Squad automated formal verification.*" (CRITIQUE.md:3); the paper's affiliation is "Automated Formal Verification Agent". (F) Research-stage proofs for a production-intent library with an AI-assisted proof upkeep loop: "agent hooks that let an AI do the model upkeep while blocking it from weakening a theorem or skipping the gate" (formal/README.md:61). |
| AI provenance | author-reported, per artefact: (V) README "written and verified in Lean 4 by Claude!" (a `CLAUDE.md` is also present); (Q) CRITIQUE.md:3 "Written by Lean Squad automated formal verification" and paper author line "Lean Squad / Automated Formal Verification Agent"; (F) formal/README.md:61 "agent hooks that let an AI do the model upkeep". (G) none-found. Not used to discount any result (CONVENTIONS). Index value: author-reported. |

## 2. Claim as stated by the authors (verbatim)
- (G) Paper abstract: "We present GhostBuster, a runtime system which monitors the execution of BGP routers and verifies their compliance with the protocol specification. Concretely, GhostBuster checks whether observed outgoing BGP messages could have been produced by incoming ones. ... one can solve this problem efficiently by reasoning about sets of messages instead of orderings." README: "`proof/` holds the machine-checked version of the paper's correctness argument."
- (V) README: "The implementation is proven against the generated propositions, so the chain from the words of the RFC to the theorem about the running code is machine-checked end to end." And: "For the covered path — recursive IN-class, non-`ANY` resolution — the implementation is proven to refine an RFC-derived model (`ioResumeLoop_sound`, axiom-clean): every answer the running server returns is one the model authorizes."
- (Q) paper.tex abstract: "Using \emph{Lean~4} without Mathlib, we have proved \textbf{1{,}567 named theorems across 74 files} ... Behavioural correspondence between the Lean functional models and the Rust source is validated by 23 Route-B test suites covering 2{,}822+ cases, all passing."
- (F) README: "The codecs and protocol state machines are [proved against the RFCs in Lean 4]". formal/README.md: "The work is a transliteration plus proofs, not a verified compiler, so the trust base below still applies."

## 3. What is actually formalised
### (G) GhostBuster
The whole proof is the single file `proof/Proof.lean`, 309 lines. The router is an abstract function, `proof/Proof.lean:19-20`:
```lean
-- We assume the spec as an axiom
axiom spec : State -> Msg
```
Here `Msg = Option (Route × ℝ)`, `State = Neighbor → Msg` and `Seq = Neighbor → List Msg`. A "witness" is a chain of per-neighbour input prefixes that explains each output (`WitnessFrom`, `ValidFrom`). `GhostBusterAlg` is a set-valued description of the algorithm that picks *minimal* prefixes (`MinStep`). Headline, `Proof.lean:292`:
```lean
theorem ghost_buster_valid_from
        (i prev : Seq) (o : List Msg)
        (ref : State -> Msg) (href : spec = ref) :
    ValidFrom prev i o ↔ GhostBusterAlg i prev o ref ≠ {} := by
```
Supporting results: `valid_minimal_witness_exists` (`:170`) and lemma `ghost_buster_witness_reaches` (`:262`). This proves the reduction "an explanation exists ⇔ a minimal-step explanation exists". It is a statement about a mathematical `Set`, not about the Rust verifier in `src/reordering_monitoring.rs`.

### (V) VeriDNS
128 Lean files, 83,393 lines. The parts are:
- An RFC-text embedding macro (`include_rfc`) plus an NLP pipeline (`VeriDNS/RFC/`) that generates `Prop` specs.
- An implementation in `VeriDNS/Impl/`: codec, cache, resolver, server.
- A network semantics/model in `Spec/NetworkSemantics.lean`, 4,674 lines.
- Proofs in `VeriDNS/Proof/`, e.g. `Refinement.lean` (9,614 lines) and `IoResumeSound.lean` (11,173 lines).

The environment is an abstract network/world, and soundness is conditional on an honesty predicate over it. `VeriDNS/Proof/NameTree.lean:1699`:
```lean
def NetworkConsistent (T : Node ResourceRecord) (M : Type → Type) (Sock : Type)
    [Monad M] [UdpSocket M Sock ByteArray] : Prop :=
  ∀ (q : Format) (addr : ByteArray),
    SatisfiesM (m := M)
      (fun ro => ∀ resp₀ resp, ro = some resp₀ →
        acceptResponse q resp₀ = some resp → ResponseConsistent T resp)
      (forwardQuery (M := M) (Sock := Sock) q addr)
```
`NameTree.lean:2018`:
```lean
theorem resolveWithIO_sound (T : Node ResourceRecord)
    (hnet : NetworkConsistent T M Sock) (hnetTcp : NetworkConsistentTcp T M Sock)
    (query : Format) (sbelt : DnsSList)
    {cache : DnsCache} (hc : CacheAgrees T cache) (now : UInt32)
    (fuel depth : Nat) (budget : UInt32) :
    SatisfiesM (ShimSound T)
      (resolveWithIO (M := M) (Sock := Sock) query sbelt cache now
        fuel depth budget)
```
The refinement capstone is `ioResumeLoop_sound` at `VeriDNS/Proof/IoResumeSound.lean:3571`, with hypotheses `StateModels`, `WorldModelsTcp`, `CacheWf`, `CacheNsCanon`, `CacheCnameCanon`, `GluelessProv`, …. Answer authenticity, `VeriDNS/Spec/AnswerAuthenticity.lean:158`:
```lean
theorem scrubAnswer_no_foreign {qname : Name} {answer : List RR} {rr : RR}
    (hrr : rr ∈ scrubAnswer qname answer)
    (hforeign : ∀ n, CnameReachable qname answer n → nameEq rr.owner n = false) : False
```
The outermost serving loops are `partial def` and so outside any theorem: `VeriDNS/Main.lean:84 udpServeLoop`, `:115 tcpConnLoop`, `:134 tcpServeLoop`, and `VeriDNS/Impl/Server.lean:1202 serverLoop`. `ioResumeLoop` (`Impl/Server.lean:864`) and `resolveWithIO` (`:1030`) are total `def`s.

### (Q) Lean Squad quiche
77 Lean files, 22,383 lines. Each file is a hand-written (agent-written) functional model of a Rust function. Example, `FVSquad/Varint.lean:7,15`: "codec in `octets/src/lib.rs`" and "Bitwise OR (|||) in the Rust encoder is modelled as addition (+)". `Varint.lean:221`:
```lean
theorem varint_round_trip (v : Nat) (hv : v ≤ MAX_VAR_INT) :
    ∃ bytes, varint_encode v = some bytes ∧ varint_decode bytes = some v
```
`FVSquad/PacketNumDecode.lean:289` `decode_pktnum_correct` is stated over `Nat` with RFC 9000 §A.3 window hypotheses. CORRESPONDENCE.md grades each model as "exact | abstraction | approximation | mismatch" relative to the Rust code. There is no environment or adversary model.

### (F) flare/formal
171 Lean files, 54,125 lines; REPORT gives "166 (53696 lines)", "2680" theorems and "730" headline theorems in the axiom audit. Each impl model is a transliteration of Mojo code carrying an anchor such as `/-- mirrors flare/quic/varint.mojo#decode_varint sha=2fbdb2fed88f at=59bda50f -/`. Specs are written separately from the RFCs. Theorems cover "refinement, round trips, invariants over every reachable state, progress, absence of overflow, and independence from how the input is split into reads" (REPORT §1). A top-level `Machine.lean` composes one reactor worker; `MachineWorkers.lean` lifts it to several workers. Environment facts are `Prop` hypotheses in `Flare/Core/Assumptions.lean`.

## 3a. Predicate semantics
(G) GhostBuster, `ghost_buster_valid_from` (`proof/Proof.lean:292`).
- Conclusion left side (`Proof.lean:60-61`, verbatim): `def ValidFrom (prev i : Seq) (o : List Msg) : Prop := WitnessReaches prev i o ∧ spec (state i) ≊ o.getLast?.join`. Right side (`:90-91`, verbatim): `def GhostBusterAlg (i : Seq) (prev : Seq) (o : List Msg) (ref : State -> Msg) : Set (List Seq) := { w ∈ GhostBusterRun i prev o ref | ref (state i) ≊ o.getLast?.join }`, where each step is a `MinStep` (`:63-67`): `prev ⊑ m ∧ ref (state m) ≊ o ∧ ∀ inbetween, prev ⊑ inbetween → inbetween ⊑ m → spec (state inbetween) ≊ o → m = inbetween`.
- Strongest premise: `href : spec = ref`, with `spec` an axiomatised constant (`:19-20`).
- Guaranteed: for an arbitrary router function, the outputs can be explained by some sequence of input prefixes if and only if the minimal-prefix search set is non-empty. Not guaranteed: anything about the Rust monitor's code, its search procedure's termination, or a real router's behaviour; `spec` is uninterpreted.

(V) VeriDNS, `resolveWithIO_sound` (`VeriDNS/Proof/NameTree.lean:2018`) and `ioResumeLoop_sound` (`VeriDNS/Proof/IoResumeSound.lean:3571`).
- `ShimSound` (`NameTree.lean:1715-1717`, verbatim): `(∀ f, rc.1 = .ok f → SectionAgrees T f.answer) ∧ CacheAgrees T rc.2`, where `SectionAgrees root rrs := ∀ b ∈ rrs.toList, RRAgrees root b` and `RRAgrees root bytes := ∃ rr, RRParse.parseRaw bytes = some rr ∧ RRInTree root rr` (`:33-38`).
- Strongest premise of `resolveWithIO_sound`: `hnet : NetworkConsistent T M Sock` (`:1699`, quoted in §3): every response returned by `forwardQuery` and accepted by `acceptResponse` is `ResponseConsistent T` (a structure at `NameTree.lean:86`, fields not quoted here). So the guarantee is relative to a network that only returns records from the fixed tree `T`.
- `ioResumeLoop_sound` conclusion (`IoResumeSound.lean:3613-3630`, verbatim excerpt): `∃ slist v coutM, HasVerdictAt net ns ra ednsBuf rttOf now nseen seen c slist q v coutM ∧ ... ∧ (αResp resp).rcode = v.rcode ∧ (αResp resp).answer = αSection state.cnameChain ++ v.answer ∧ CacheRefines (αCache cout) coutM ∧ WorldModels net ns ra ednsBuf now w' ∧ ...`. `HasVerdictAt` (`Proof/Refinement.lean:2780-2787`) is `∃ tr sp tEnd resp, Resolves net ns ... q tr sp tEnd coutM resp ∧ RespAgree v resp`. Premises (`:3581-3612`) include `StateModels`, `WorldModelsTcp`, cache well-formedness families, `q.rd = false`, `q.qtype ≠ QType.star`, `q.qclass = RRClass.in`, `state.now.toNat + 604800 < 2 ^ 32`, name length bounds, and a successful `Prog.run n (Server.ioResumeLoop ...) w = some ((.ok resp, cout), w')`.
- Guaranteed: when the resume loop returns a response for a recursive IN-class non-ANY query and the world model hypotheses hold, the rcode and answer agree with some run of the RFC-derived `Resolves` model, and the cache still refines the model cache. Not guaranteed: anything for the `partial` serve loops or FFI sockets; that `WorldModels`/`StateModels` are satisfiable by a non-vacuous initial state (VeriDNS's own audit says no such witness lemma exists, S4).

(Q) quiche, `varint_round_trip` (`FVSquad/Varint.lean:221`).
- Conclusion: `∃ bytes, varint_encode v = some bytes ∧ varint_decode bytes = some v`. `varint_encode` (`Varint.lean:69-79`, verbatim start): `def varint_encode (v : Nat) : Option (List Nat) := if v ≤ 63 then some [v] else if v ≤ 16383 then let w := v + 16384; some [w / 256, w % 256] else ...`. Premise: `hv : v ≤ MAX_VAR_INT`.
- Guaranteed: the agent-written `Nat` model of the encoder and decoder round-trips on the valid range. Not guaranteed: anything about the Rust `octets` code (bytes are `Nat`, `|||` is modelled as `+`; correspondence is by Route-B tests), or fixed-width overflow behaviour (paper.tex §Limitations, "Overflow not verified").

(F) flare, `decode_encode` (`formal/Flare/L1_Encoding/QuicVarint.lean:82-83`).
- Conclusion (verbatim): `∃ e, encode v = some e ∧ decode (e ++ rest) = some (v, e.length)` for `v : UInt64`, `h : v ≤ VARINT_MAX`, any `rest : Bytes` (`Bytes := List UInt8`, `Flare/Core/Bytes.lean:9`). `encode` carries the anchor `/-- mirrors flare/quic/varint.mojo#encode_varint+encode_varint_into sha=bb7358d4e5a1 at=f52cce73 -/` (`:39`).
- Guaranteed: the Lean transliteration round-trips with any suffix and reports consumed length, on fixed-width `UInt64`. Not guaranteed: that the Mojo code equals the transliteration (the anchor checks the source hash has not changed since transliteration; equality is checked by differential tests, not proved).

## 4. Property classes covered
| Class | (G) GhostBuster | (V) VeriDNS | (Q) quiche | (F) flare |
|---|---|---|---|---|
| Functional correctness | proved (algorithm-level set characterisation) | proved (treeLookup vs. RFC 1034 §4.3.2; cache discipline; negative caching) | partial (pure functional models, `Nat` arithmetic, happy path) | proved (per-component impl-vs-spec) |
| Safety invariant | no | proved (cache invariant pack; bailiwick; credibility) | partial (e.g. `emitN_le_maxData`, BBR2 guards) | proved ("invariants over every reachable state") |
| Liveness | no | partial (`docs/liveness-plan.md`; adequacy/no-starvation results; not read in full) | no | partial ("progress", "connection state machine (with liveness …)") |
| Refinement | no | proved, conditional (`ioResumeLoop_sound`: impl refines RFC-derived model) | no (correspondence by tests only) | partial (HTTP/2 stream states vs RFC 9113 §5.1) |
| Symbolic / computational security | no | partial: off-path spoofing resistance under a modelled world with a `SpoofReply` arm; no crypto | no | no (OpenSSL/rustls are black boxes) |
| Parsing | no | proved (`decodeName` bounds, compression-pointer termination, `decode_encode_of_decode`) | proved (varint, octets, packet header, H3 frames, QPACK integer round trips) | proved (codecs, HPACK Huffman, QUIC frame parser) |
| Constant-time | no | no | no | no |

## 5. Link to implementation code
- (G) None mechanised. The Rust verifier (`src/`) is evaluated empirically (simulated networks; GNS3 with real FRR routers). The Lean proof is about the algorithm.
- (V) Lean is the implementation. The resolver core, including the IO resume loop, is the proved code, parameterised by a monad `M` and a `UdpSocket` class. The sockets, clock and RNG are 14 `@[extern]` C functions (`VeriDNS/Impl/UdpSocket.lean:6-46`), and the outer serve loops are `partial`.
- (Q) The Rust code is the implementation, and the Lean models are hand-written by the agent. The link is "Route-B" executable correspondence tests: "23 Route-B test suites covering 2,822+ cases" per paper.tex; REPORT says "Twenty-three targets … (1,759+ cases)"; CRITIQUE says "27 Route-B test targets (2864+ PASS)".
- (F) The Mojo code is the implementation, and the Lean models are transliterations. The link has three parts: source-hash anchors checked per commit (`scripts/anchors.py`, `tiers.toml`), a statement lock (`spec.lock`, re-lock only by a human), and differential tests in which the Lean model is compiled to `flare-oracle` and run against the real Mojo functions (`difftest/run.py`).

## 6. Trusted computing base and soundness hygiene
Static token counts, comments with `--` stripped (block/doc comments not stripped). Real declarations are noted separately.
| Item | (G) | (V) | (Q) | (F) |
|---|---|---|---|---|
| files / LOC | 1 / 309 | 128 / 83,393 | 77 / 22,383 | 171 / 54,125 |
| theorem/lemma decls | 20 | 2,868 | 1,592 | 2,695 |
| sorry / admit | 0 / 0 | 0 / 0 | 0 / 0 | 0 / 2 tokens (in prose) |
| axiom | 1 real: `axiom spec : State -> Msg`, an opaque data constant of an inhabited type (equivalent to a section variable; introduces no inconsistency) | 2 tokens, both in prose; 0 declarations | 0 | 10 tokens, all in prose; 0 declarations (REPORT §3.5: "There are no `sorry`, `admit` or `axiom` declarations") |
| native_decide | 0 | 0 (roadmap: "`native_decide` ×3 … **ELIMINATED**"); codec relies on `bv_decide` SAT-certificate axioms per `docs/assurance-roadmap.md` GAP 4 | 260 tokens | 0 ("`native_decide` is not allowed") |
| extern / opaque | 0 / 0 | 14 `@[extern]` / 16 opaque tokens (FFI sockets, clock, RNG) | 0 / 3 | 0 / 2 |
| partial | 0 | 5 `partial def` (4 serve loops + 1 RFC-text parser) | 0 | 1 `partial def` (`Core/TrustSurface.lean:20`, a meta tool) |

- (V) self-audit: the repo contains its own audit with severity-ranked findings against its theorems. `docs/audit/w4-summary.md`, row 2: "**`WorldModels`/`WorldModelsTcp` are never discharged for any world — satisfiability of the whole soundness spine is unpinned.** ... Only the vacuous `oracle = fun _ _ => none` world satisfies the pack trivially." Row 1: "**`primeRootHints` breaks every serve capstone at the real entry point.**" Row 3: "**`Resolves.gaveUp` is premise-free, so every SERVFAIL 'verdict' conjunct is content-free.**" The audit is dated 2026-07-15 and the pinned commit is 2026-07-17. Whether these rows were closed by that commit was not determined.
- (Q): 260 `native_decide` tokens put trust in the Lean compiler for those goals. The models use `Nat` for Rust `u64`/`u8` (see §8).
- (F): `formal-check` "fails on any `sorry`, `admit`, `axiom` or `native_decide`, and on any audited theorem whose axioms go beyond" `propext`, `Quot.sound`, `Classical.choice` (REPORT §1).

## 7. Scale, effort, automation
- (G): 309 lines, 20 lemmas/theorems, Mathlib. README: "Checking the Lean proof, after `lake exe cache get` | ~30 s".
- (V): 83k lines, 2,868 theorems; about 7.8k lines of planning/audit docs in `docs/`. Machine-generated (Claude). The docs site shows per-RFC coverage, e.g. RFC 5452 "257 / 259 lines formalized (99.2%)" (search snippet of the docs site, not opened; see §15).
- (Q): 22k lines, 1,592 theorem declarations by grep. The documents report 1,348 (CRITIQUE, run 177), 1,443 (REPORT, 2026-05-22) and 1,567 (paper). Runs are numbered past 182. paper.tex: "The median proof is 3--8 lines: a combination of \texttt{simp} to unfold definitions and \texttt{omega}". "The five-phase pipeline runs autonomously across CI".
- (F): 54k lines, 2,680 theorems, 138 findings (6 high, 50 medium, 81 low, 1 info), with "138 of 138" resolved per REPORT §1. Tooling: anchors, spec lock, difftest, a trust-surface tool, Cursor agent hooks.

## 8. Limitations stated by the authors (verbatim)
- (G) README: the GNS3 experiments "drive real FRR routers in an emulated network, so **the numbers are not deterministic.**" The paper abstract states detection of "over 60% of the bugs" (i.e. incomplete detection), with "never falsely reports a bug" on simulated networks.
- (V) README: "**DNSSEC validation.** Records are not cryptographically authenticated." "**TCP transport and TCP fallback.** UDP only." "**Concurrent serving** (single-threaded request loop)." "Only the FFI transport and the real network sit outside — that is where the honesty oracle is assumed rather than proven". `docs/assurance-roadmap.md` GAP 4: "Residual TCB = FFI + (codec's bv_decide SAT certificate) + (model ≈ RFC, human judgment)."
- (Q) paper.tex §Limitations: "All integer types are modelled as \texttt{Nat}; u64/u8 overflow is not checked." "The specification covers the ``happy path'' only." "The \texttt{tokio-quiche} async layer is not modelled." "The BoringSSL FFI layer (\texttt{ffi.rs}), TLS callbacks, and the \texttt{transmute} in \texttt{gso.rs} are not modelled."
- (F) formal/README.md: "A transliteration can be wrong in ways no proof notices, because the proof is about the model, not the code." REPORT §1: "The kernel, libc, OpenSSL and the Mojo runtime are not modelled."

## 9. Independent evidence
- (G): accepted at SIGCOMM 2026 (NSG publications page; NSG news item of 12 May 2026). Real bugs: the paper abstract says "detect (confirmed and previously unknown) bugs in production routers". The README lists experiment binaries `lp_bug`, `mrai_bug`, `pl_bug` against FRR.
- (V): no third-party review found. The project's own audit lists severity-5 gaps (§6).
- (Q): reports "a formal proof that \texttt{StreamPriorityKey::cmp} violates the \texttt{Ord} antisymmetry contract, confirmed intentional by the implementors" (paper.tex:73), via theorem `cmpKey_incr_incr_not_antisymmetric` (REPORT.md:447-453). No external maintainer statement was read. The repo is a fork, not cloudflare/quiche.
- (F): self-reported 138 findings, each fixed with a regression test in `tests/` (REPORT §1; `formal/report/Fixed.md` not read). Examples of high-rated findings: "ENC-03: an 11-byte unauthenticated gRPC health request aborts the server", "H2-01: the HTTP/2 connection-level receive window is never enforced". These were not independently confirmed here.

## 10. Reproducibility
Not built: sandbox disk limit; queued for CI reproducibility job. An attempt to fetch the GhostBuster toolchain was killed when the shared disk hit ENOSPC; it was not retried, per instruction. Exact commands:
- (G) `cd ghostbuster/proof && lake exe cache get && lake build` (README: Mathlib cache "7.3 GB on disk once unpacked"; check "~30 s").
- (V) `cd VeriDNS && lake build` (also builds Verso-literate docs dependencies).
- (Q) `cd quiche/formal-verification/lean && lake build`.
- (F) `cd flare/formal && lake build` (or `pixi run -e formal formal-build` and `formal-check` for the axiom audit).

## 11. Generalisation
Four distinct patterns appear:
1. (G) Algorithm-level proof of a runtime-monitoring reduction. The implementation is in Rust, is unlinked to the proof, and is validated empirically. Cheap (309 lines); assurance covers the idea, not the code.
2. (V) "Lean is the implementation" plus RFC-text-to-`Prop` generation, IO parameterised over a monad with an assumed honest/adversarial world. Preconditions: writing the server in Lean; accepting world-model hypotheses that must be shown satisfiable (the project's own audit flags that they were not).
3. (Q) An agent writes functional models of mainstream-language (Rust) functions, proves lemmas about them, and links them to the code with executable correspondence tests. Applies to arbitrary codebases. The models abstract `uN` as `Nat`, and correspondence is tested, not proved.
4. (F) Transliteration with maintenance machinery: source-hash anchors, a human-only statement lock, differential testing of the Lean oracle against shipped code, and "verified kernels" that route critical decisions through small sans-I/O functions. Applies to non-Lean codebases (Mojo here). It requires the code to be factored into sans-I/O pieces and a person to own the spec lock.

## 12. Bearing on sub-claims (facts only, no verdict)
| Sub-claim | Fact from this reference | Source ID |
|---|---|---|
| H1 | DNS resolution with an adversarial off-path world (V); BGP reordering semantics (G); QUIC/HTTP/3 codecs and congestion-control arithmetic (Q, F); HTTP/2 stream states and multi-worker reactor (F). All are expressed in Lean. Cryptography is out of scope in all four. | S2, S3, S5, S7 |
| H2 | All four report 0 `sorry`. (V) and (F) report axiom-clean headline theorems. (V)'s own audit says its soundness spine's world hypotheses have no satisfying non-vacuous witness. (Q) uses 260 `native_decide`. None was re-checked here. | S3, S4, S5, S7 |
| H3 | Lean-is-impl (V); hand/agent model plus executable tests (Q); transliteration plus anchors plus difftest (F); no link (G). | S2, S3, S5, S7 |
| H4 | (Q) and (F) apply Lean to Rust and Mojo codebases via models. Both state that the model can diverge from the code (`Nat` vs `u64` in Q; "transliteration can be wrong" in F). | S5, S6, S7 |
| H5 | (F) reports 138 code bugs found via failed proofs; (Q) reports an `Ord`-contract violation; (G) finds router bugs empirically, not via the Lean proof (paper abstract, §15 lead). | S2, S6, S7 |
| D | (V) "end to end" means RFC prose → NLP-generated `Prop` → theorem about Lean code, excluding FFI and outer loops. (F) "proved against the RFCs" means statements about transliterated models. (Q) "proved" means theorems about agent-written models. | S3, S5, S7 |

## 13. Conflicts
- (V) The README scope says "**TCP transport and TCP fallback.** UDP only" and "**Concurrent serving** (single-threaded request loop)". However `Main.lean` has `tcpConnLoop`/`tcpServeLoop` and `Std.Mutex DnsCache`, `UdpSocket.lean` has `veri_dns_tcp_listen`/`_accept`, and theorems carry `NetworkConsistentTcp`/`WorldModelsTcp`. The README scope text appears to predate the TCP work.
- (V) The README describes `ioResumeLoop_sound` as giving that "every answer the running server returns is one the model authorizes". The repo's audit (`docs/audit/w4-summary.md`) lists the unwitnessed `WorldModels` pack (sev 5) and `primeRootHints` breaking capstones "at the real entry point" (sev 5).
- (Q) Theorem totals differ across documents (1,348 / 1,443 / 1,567, against 1,592 by grep), as do Route-B totals (1,759+ / 2,822+ / 2,864+). These are snapshots from different runs.
- (Q) Toolchain: `lean-toolchain` is `v4.29.0`; REPORT says "Lean 4.29.1".
- Name: "VeriDNS" also names a non-Lean DNS-configuration verifier (Du et al., *Computer Networks* 2026).

## 14. Sources
| ID | Title | URL | P/S | Depth | Summary |
|---|---|---|---|---|---|
| S2 | nsg-ethz/ghostbuster README and `proof/Proof.lean` @ 7564d6e, accessed 2026-11-07 | https://github.com/nsg-ethz/ghostbuster | P | [CODE] | 309-line Mathlib proof that a valid explanation exists iff the minimal-step set algorithm is non-empty; `spec` is an opaque constant. |
| S3 | BasisResearch/VeriDNS README and source @ 74bb5f3, accessed 2026-11-07 | https://github.com/BasisResearch/VeriDNS | P | [CODE] | Claude-written Lean DNS resolver; RFC-embedded NLP-generated specs; `resolveWithIO_sound`, `ioResumeLoop_sound`, `scrubAnswer_no_foreign`; FFI sockets; partial serve loops. |
| S4 | VeriDNS `docs/audit/w4-summary.md` and `docs/assurance-roadmap.md` @ 74bb5f3, accessed 2026-11-07 | https://github.com/BasisResearch/VeriDNS/tree/main/docs | P | [SECT] | Sections read: w4-summary header, severity legend and first 11 ranked rows; assurance-roadmap "GAP 4 — the TCB". Quote: "Severity: 5 = capstone-invalidating / real-system hole, 4 = major scope or vacuity gap". Self-audit ranking gaps (unwitnessed world models, unsanitised root priming, premise-free give-up rule); TCB inventory. |
| S5 | dsyme/quiche `formal-verification/` (REPORT.md, CRITIQUE.md, CORRESPONDENCE.md headers, paper/paper.tex, lean/) @ 5a8195e, accessed 2026-11-07 | https://github.com/dsyme/quiche/tree/main/formal-verification | P | [CODE] | Agent-written Lean models of quiche functions; 1,592 theorem decls; Route-B tests; limitations (Nat for uN, happy path only). |
| S6 | Lean Squad paper.tex, "Incremental Formal Verification of a Production QUIC Implementation with Lean 4", @ 5a8195e, accessed 2026-11-07 | https://github.com/dsyme/quiche/blob/main/formal-verification/paper/paper.tex | P | [SECT] | Sections read: abstract, \subsection{Limitations} (`paper.tex:654`), \subsection{Automation and Effort} (`:684`); middle sections not read. Quote: "All integer types are modelled as \texttt{Nat}; u64/u8 overflow is not checked." |
| S7 | ehsanmok/flare `formal/README.md`, `formal/REPORT.md` (§1, §3.5 header) and source @ 576799d, accessed 2026-11-07 | https://github.com/ehsanmok/flare/tree/main/formal | P | [CODE] | Transliterated Lean models of Mojo networking code; anchors, spec lock, difftest; 2,680 theorems; 138 fixed findings; no sorry/axiom/native_decide policy. |

## 15. Unverified leads (excluded from evidence)
- NSG ETH publication page, "When static verification is not enough: revealing BGP bugs at runtime" (SIGCOMM 2026), https://nsg.ee.ethz.ch/publications/2026-05-13-when-static-verification-is-not-enough-revealing-bgp-bugs-at-runtime-20-500-11850-800202/ . Abstract only (runtime checking of BGP outputs against inputs under reordering; set-based reasoning; confirmed bugs in production routers; >60% detection). The paper itself was not read; abstract statements in §2 rest on this page.
- GhostBuster arXiv version (listed on NSG publications page; not opened).
- VeriDNS docs site per-RFC coverage, e.g. https://basisresearch.github.io/VeriDNS/source/rfc-5452.html ("257 / 259 lines formalized (99.2%)", seen in a search snippet only).
- Du, Hu, Wang, Guo, "VeriDNS: incremental distributed verification of DNS configurations", *Computer Networks* 275 (2026) 111929, doi:10.1016/j.comnet.2025.111929. A different, non-Lean project, seen only as a citation in the ACM GRoot/SemDNS pages.
- flare `formal/report/Fixed.md` (per-finding fix commits) and the `formal-audit-0.12` git tag (not read).
- Whether cloudflare/quiche maintainers confirmed the `StreamPriorityKey` behaviour (paper.tex:486 says so; no primary maintainer statement read).
