# REF-C05: Tamarin case studies (TLS 1.3, 5G AKA, EMV), with the Tamarin-to-code bridge of Arquint et al.

Kind: comparator (non-Lean). Access date for all sources: 2026-11-07. Static inspection only; no tool was installed or run (see Section 10).

## 1. Identity
| Field | Value |
|---|---|
| Subject (protocol or algorithm) | Three symbolic (Dolev-Yao) protocol models verified with the Tamarin prover: (a) TLS 1.3 draft 21 handshake, all modes; (b) 3GPP 5G AKA authentication; (c) EMV chip payment, contact and contactless (Visa, Mastercard kernels). Plus (d) Arquint et al. (S&P 2023), the published route from a Tamarin model to verified Go/Java code, demonstrated on WireGuard. |
| Spec it targets | (a) draft-ietf-tls-tls13-21 (pre-RFC 8446), Appendix E.1 properties [C05-S1]. (b) 3GPP TS 33.501 v15.0.0/v15.1.0 plus TS 33.102, TR 33.899 ("722 pages across 4 documents") [C05-S3, C05-S4]. (c) EMV Books plus Visa and Mastercard contactless kernel specs ("over 2,000 pages") [C05-S5]. (d) WireGuard key exchange as implemented in wireguard-go [C05-S8]. |
| Organisation / authors | (a) Cremers, Horvat, Hoyland, Scott, van der Merwe (Oxford, MPI-SWS, Royal Holloway), ACM CCS 2017. (b) Basin, Dreier, Hirschi, Radomirović, Sasse, Stettler (ETH Zurich, LORIA, Dundee), ACM CCS 2018. (c) Basin, Sasse, Toro-Pozo (ETH Zurich), IEEE S&P 2021. (d) Arquint, Wolf, Lallemand, Sasse, Sprenger, Wiesner, Basin, Müller (ETH Zurich, IRISA), IEEE S&P 2023 (arXiv 2212.04171). |
| Primary URLs | https://tls13tamarin.github.io/TLS13Tamarin/docs/tls13tamarin-draft21.pdf ; https://arxiv.org/abs/1806.10360 ; https://arxiv.org/abs/2006.08249 ; https://arxiv.org/abs/2212.04171 ; https://tamarin-prover.com/manual/master/book/014_limitations.html |
| Repo, commit SHA, last commit date, licence | (a) github.com/tls13tamarin/TLS13Tamarin @ 67262370672b2cbcc461179e9d3e87eddcfcb999, 2023-08-19; no LICENSE file in repo. (b) github.com/tamarin-prover/tamarin-prover, `examples/ccs18-5G/` @ 9228c58d987831881eb5916310861274e99ca48d, 2026-11-06; GPL-3.0 (repo licence). (c) github.com/EMVrace/EMVerify @ 592200990f6738f41c540f9c619e11e932b0c6d5, 2021-02-08; no LICENSE file in repo. (d) Code referenced as [20] in the paper; not cloned (disk limit). |
| Tool and version | Tamarin prover (Haskell, Maude backend). Versions used by the papers: (b) "Tamarin v1.4.0" (needed for XOR) [C05-S3, line 926]; (c) "Tamarin v1.5.1" [C05-S5, Table I caption]; (a) version not stated in the paper. Current repo HEAD declares `version: 1.13.0` (tamarin-prover.cabal:4). (d) Tamarin + Gobra (Go) + VeriFast (Java). |
| Maturity | Tamarin: mature research tool, actively maintained (commit dated the day before access). Case-study models: research artefacts, frozen at publication (TLS 2023, EMV 2021). AI provenance: `none-found`. Basis: no AGENTS.md, CLAUDE.md, .claude, .cursor or Copilot config in the TLS13Tamarin or EMVerify trees at the recorded SHAs, and no AI co-author trailers. These were depth-1 clones, so only the HEAD commit was checked; the tamarin-prover tree was not re-checked. The artefacts predate 2022 (TLS 2017, 5G 2018, EMV 2021) and are peer-reviewed. |

## 2. Claim as stated by the authors (verbatim)
- TLS 1.3: "We further this effort by constructing the most comprehensive, faithful, and modular symbolic model of the TLS 1.3 draft 21 release candidate, and use the Tamarin prover to verify the claimed TLS 1.3 security requirements, as laid out in draft 21 of the specification. In particular, our model covers all handshake modes of TLS 1.3." [C05-S1, lines 22-26]
- TLS 1.3: "Our analysis reveals an unexpected behaviour, which we expect will inhibit strong authentication guarantees in some implementations of the protocol." [C05-S1, lines 27-28]
- 5G AKA: "We provide the first comprehensive formal model of a protocol from the AKA family: 5G AKA. ... Our automated analysis identifies the minimal security assumptions required for each security goal and we find that some critical security goals are not met, except under additional assumptions missing from the standard." [C05-S3, lines 51-59]
- EMV: "We formalize a comprehensive symbolic model of EMV in Tamarin ... We use our model to automatically identify flaws that lead to two critical attacks: one that defrauds the cardholder and a second that defrauds the merchant." [C05-S5, lines 15-20]
- Arquint et al.: "We automatically translate a Tamarin protocol model into a set of I/O specifications expressed in separation logic. ... Our soundness result guarantees that the verified implementation inherits all security (trace) properties proved for the Tamarin model." [C05-S8, abstract]

## 3. What is actually formalised

### 3.1 Common modelling substrate (Tamarin)
- Protocol = multiset rewriting rules `[premises] --[actions]-> [conclusions]` over facts; global state is a multiset of linear and persistent facts; traces are sequences of action multisets; properties are first-order formulas over traces with timepoints [C05-S1, lines 470-500; C05-S5, lines 533-609].
- Adversary: Dolev-Yao, "complete control of the network, and can intercept, send, replay, and delete any message"; "perfect cryptography" [C05-S1, lines 730-735]. Cryptography is an equational theory over terms ("black box cryptography assumption") [C05-S3, lines 834-842].
- Unbounded sessions: "We focus on ruling out complex interaction attacks by considering an unbounded number of concurrent connections" [C05-S1, line 1262]; 5G: "unboundedly many instances of each role" [C05-S3, line 884]; EMV: "unboundedly many concurrent protocol sessions" [C05-S5, line 77].

### 3.2 TLS 1.3 draft 21 (repo `src/rev21`)
- Model: m4 macro files `model/*.m4i` (client_basic, client_psk, server_basic, server_psk, post_hs, record, zero_rtt, pki, crypto, msgs, state, adversary; 2,129 lines total). Lemma files `lemmas/*.m4` (959 lines). A basic handshake "invokes up to 21 different rules" [C05-S1, line 547].
- Key schedule is transcribed from the draft via macros, e.g. `define(<!HandshakeSecret!>, <!Extract(gxy, es)!>)` and `define(<!resumption_master_secret!>, <!Derive_Secret(ms, 'TLS13resumptionmastersecret', ...)!>)` (`model/crypto.m4i`, lines ~25-33).
- Adversary compromise rules, verbatim (`model/adversary.m4i:1-9`):
```
rule Reveal_Ltk:
  [ !Ltk($A, ~ltkA) ] --[ RevLtk($A) ]-> [ Out(~ltkA) ]
rule Reveal_DHExp:
  [ DHExp(~x, ~tid, $A) ] --[ RevDHExp(~tid, $A,~x) ]-> [ Out(~x), DHExp(~x, ~tid, $A) ]
rule Reveal_PSK:
    [ SecretPSK($A, res_psk)]--[ RevealPSK($A, res_psk) ]->[Out(res_psk)]
```
- Main secrecy lemma, verbatim (`lemmas/lemma.m4:174-182`; also printed as Figure 5 of the paper):
```
lemma_secret_session_keys/*:
  "All tid actor peer kw kr pas #i.
      SessionKey(tid, actor, peer, <pas, 'auth'>, <kw, kr>)@i &
      not (Ex #r. RevLtk(peer)@r & #r < #i) &
      not (Ex tid3 x #r. RevDHExp(tid3, peer, x)@r & #r < #i) &
      not (Ex tid4 y #r. RevDHExp(tid4, actor, y)@r & #r < #i) &
      not (Ex rms #r. RevealPSK(actor, rms)@r) &
      not (Ex rms #r. RevealPSK(peer, rms)@r)
        ==> not Ex #j. K(kr)@j"
```
- Unilateral authentication, verbatim (`lemmas/lemma.m4:241-249`):
```
lemma_entity_authentication/* [reuse, use_induction]:
  "All tid actor peer nonces cas #i.
      commit(Nonces, actor, 'client', nonces)@i & commit(Identity, actor, 'client', peer, <cas, 'auth'>)@i &
      not (Ex #r. RevLtk(peer)@r & #r < #i) & ... RevDHExp ... RevealPSK ...
          ==> (Ex tid2 #j. running2(Nonces, peer, 'server', nonces)@j & #j < #i)"
```
- Session-key agreement (`lemmas/lemma.m4:326-334`): `SessionKey(tid, actor, peer, as, <kr, kw>)@i & running(Nonces, actor, 'client', nonces)@j & SessionKey(tid2, peer, actor, as2, <kw2, kr2>)@k & running2(Nonces, peer, 'server', nonces)@l ==> kr = kr2 & kw = kw2`.
- Other main lemmas in the same file: `secret_session_keys_pfs` (:199), `unique_session_keys` (:211), `transcript_agreement` (:255), `mutual_entity_authentication` (:269), `mutual_transcript_agreement` (:283), `mutual_injective_entity_authentication` (:297). Stored-proof summary `proofs/summaries.txt` records all as "verified", e.g. "secret_session_keys (all-traces): verified (372 steps)", "injective_mutual_entity_authentication (all-traces): verified (2651 steps)".
- Attack artefact: `attack/auth_mismatch.m4`, `attack/MAA-contradiction.spthy` (client/server authentication-status mismatch, Section 5.2 of the paper).

### 3.3 5G AKA (tamarin-prover `examples/ccs18-5G/`)
- Three model families: `5G-AKA-nonBindingChannel` (TS 33.501 v15.0.0 as written), `5G-AKA-bindingChannel` (adds binding SN-HN channel assumption), `5G-AKA-untraceability` (observational-equivalence privacy) (README.md).
- `5G-AKA-nonBindingChannel/5G_AKA.spthy`: 2,771 lines including comments, 25 rules, 80 `lemma` declarations; builtins `asymmetric-encryption, multiset, xor` (line 129-130); functions f1..f5, f1_star, f5_star declared as uninterpreted MAC/KDF symbols (lines 132-140). Oracle `5G_AKA.oracle` 661 lines. Paper: "The protocol model itself consists of roughly 500 LoC"; "124 different lemmas and ca. 1000 LoC"; proof strategies "∼1000 LoC of Python" [C05-S3, lines 1021, 1046, 1054]. (Line counts differ from the paper's; the paper's counts likely exclude comments and span both channel variants; not resolved.)
- SQN counters modelled as multiset-encoded natural numbers; freshness check as natural-number comparison; XOR concealment modelled with Tamarin's XOR theory [C05-S3, lines 904-914].
- Example lemma found violated (stored attack), verbatim (`5G-AKA-nonBindingChannel/5G_AKA.spthy:2329-2332`):
```
lemma noninjectiveagreement_seaf_ue_kseaf_keyConf_noRev [heuristic={tactic_4}, hide_lemma=sqn_ue_nodecrease, hide_lemma=sqn_ue_src, hide_lemma=sqn_hss_src]:
	" All a b t #i. CommitConf(a,b,<'SEAF','UE',<'K_seaf', t>>)@i
    		==> (Ex #j. Running(b,a,<'SEAF','UE',<'K_seaf', t>>)@j)
             	    | (Ex X data #r. Rev(X,data)@r & Honest(X)@i) "
```
- Example lemma marked "proof (automatic) (~1 sec)" (`:2335-2339`): `noninjectiveagreement_seaf_ue_keyConf_supi_noKeyRev_noChanRev`.
- Secrecy lemma, verbatim (`:2702-2706`): `All idSN t #i. Secret(<'SEAF', idSN>, 'key', t)@i ==> not (Ex #j. K(t)@j) | (Ex X #r. Rev(X, 'secureChannel')@r & Honest(X)@i) | (Ex X key #r. Rev(X, <'k',key>)@r & Honest(X)@i)`.
- Minimality method: "we show the minimality of this set of assumptions ... by disproving all versions of the previous lemma where the set of assumptions is reduced by just one assumption" [C05-S3, lines 1050-1054].

### 3.4 EMV (EMVerify)
- Two generic models, `Contact.spthy` (907 lines) and `Contactless.spthy` (1,025 lines), plus oracles (35-44 lines each); a Makefile generates 40 "target models" per configuration (kernel, ODA method, CVM, value) (README.md). Three roles: card, terminal, bank; terminal-card channel Dolev-Yao, bank-terminal "a secure channel that offers authentication and secrecy" [C05-S5, lines 773-778].
- Properties, verbatim (`Contactless.spthy:935-950`):
```
lemma bank_accepts:
  "All t #i.
    TerminalAccepts(t)@i
   ==>
    not (Ex #j. BankDeclines(t)@j) |
    Ex A #k. Honest(A)@i & Compromise(A)@k"

lemma auth_to_terminal_minimal: //non-injective agreement with one card session
  "All T P r t #i.
    (All #a #b. OneCard()@a & OneCard()@b ==> #a = #b) &
    Commit(T, P, <r, 'Terminal', t>)@i
   ==>
    (Ex #j. Running(P, T, <r, 'Terminal', t>)@j) |
    Ex A #k. Honest(A)@i & Compromise(A)@k"
```
  plus injective `auth_to_terminal`, `auth_to_bank` (:953-996), and `secrecy_MK`, `secrecy_privkCard`, `secrecy_PIN` (:997-1012). Comment in source: "//auth_to_terminal_minimal fails for Visa EMV and for Visa DDA Low".
- Abstraction visible in source: `CTQ = CVM//encoded this way for simplicity` (`Contactless.spthy:616`).
- Stored result for Visa EMV High (`models-n-proofs/contactless/Visa_EMV_High.proof`, ~line 5500): "bank_accepts (all-traces): verified (308 steps)", "auth_to_terminal_minimal (all-traces): falsified - found trace (11 steps)", "auth_to_bank_minimal (all-traces): falsified - found trace (11 steps)", "secrecy_PAN (all-traces): falsified - found trace (7 steps)", "processing time: 75.214749429s".

### 3.5 Arquint et al. (Tamarin to code)
- Theorem 2 (Soundness): "Suppose Assumption 1 holds and that we have verified, for each role i, the Hoare triple ⊢π′ext {ψi(rid)} ci(rid) {true}. Then (|||i,rid πint(Ci(rid))) ‖Λ′ E ≼t R." [C05-S8, Section 5.2]. Consequence: "any trace property Φ proven for the protocol model, i.e., Trt(R) ⊆ Φ, is inherited by the implementation." [C05-S8, Section 3]
- Assumption 1 (Verifier assumption): "⊢α {φ} c {true} ⟹ α(C) ≼ φ." Assumption 2 (Cryptographic operations): γ injective on names, image disjointness and pattern injectivity for cryptographic function symbols [C05-S8, Sections 4.1, 4.3.3].

## 4. Property classes covered
| Class | Status | Evidence |
|---|---|---|
| Functional correctness | partial | Only "executability"/reachability sanity lemmas (EMV Definition 4 [C05-S5]; TLS `tests/reachability.m4`). No functional spec of code. |
| Safety invariant | yes (as trace properties) | Helper invariants, e.g. SQN monotonicity "strictly increasing (respectively monotonically increasing)" proved by induction [C05-S3, lines 1014-1019]; TLS `tid_invariant`, `invariant_post_hs`. |
| Liveness | no | Tamarin trace properties are prefix-closed safety-style formulas in these studies; none of the three papers states a liveness property. |
| Refinement | no (case studies); yes (Arquint) | Arquint Theorem 2 is trace inclusion implementation ≼t model [C05-S8]. |
| Symbolic security | yes | Secrecy, PFS, KCI, Lowe agreement levels (TLS Table 1 [C05-S1]; 5G Tables 1-3 [C05-S3]; EMV Tables I-II [C05-S5]). Privacy as observational equivalence in 5G [C05-S3, lines 852-858, 1369-1393]. |
| Computational security | no | "We assume perfect cryptography" [C05-S1, line 734]; downgrade "does not accurately capture the spirit ... due to the fact that we assume all cryptographic primitives are perfect" [C05-S1, lines 914-916]. |
| Parsing | no (case studies); partial (Arquint) | 5G: "we omit message bit lengths" [C05-S3, line 980]. Arquint requires formats proved "unambiguous and non-overlapping" as proof obligations [C05-S8, Section 4]. |
| Constant-time | no | Not addressed in any source read. |

## 5. Link to implementation code
- TLS 1.3, 5G AKA, EMV case studies: **none**. The models are of standards documents. EMV: "Note that our focus is on EMV's design, not implementations themselves." [C05-S5, line 132]. 5G: the paper analyses TS 33.501 versions; no implementation is mentioned in the model description. TLS 1.3: relation to the spec is via a hand-annotated RFC website ("a fully annotated version of the specification that clarifies what protocol elements we modelled") [C05-S1, lines 29-32]; that is a manual spec-to-model mapping, not a model-to-code link.
- Indirect empirical link in EMV: the model's attack trace was turned into a working attack against real terminals with a custom Android app ("We have successfully tested our PIN bypass attack on real-world terminals") [C05-S5, lines 106-111]; model ambiguities were "cross-checking and disambiguating its statements with data from over 30 real-world transaction logs" [C05-S5, lines 294-296]. This is manual, test-style validation, not a formal guarantee.
- Arquint et al.: **manual code annotation + automatic spec generation + separation-logic verification**. Tamarin model → (tool) per-role I/O specifications → existing code annotated and verified in Gobra (Go) or VeriFast (Java). Guarantee: trace inclusion (Theorem 2), so Tamarin trace properties transfer, conditional on Assumptions 1-2, on trusted library stubs, and on the code changes listed in Section 8 [C05-S8].

## 6. Trusted computing base and soundness hygiene
- Lean-specific counters (sorry/axiom/native_decide etc.) are not applicable. Tamarin analogues:
  - Tamarin itself (Haskell + Maude) is trusted; there is no independent proof-checker kernel. Stored proofs are re-checked by re-running Tamarin ("Once the proofs have been produced, they can be verified in the space of about a day, although still requiring a vast amount of RAM") [C05-S1, lines 1038-1040].
  - Tamarin manual: "the observational equivalence mode currently only (soundly) approximates observational equivalence by requiring a strict one-to-one mapping between rules" [C05-S6].
  - User-supplied oracles/heuristics are stated to be sound ("the user can supply heuristics called oracles to guide the proof search in a sound way") [C05-S3, lines 862-863]; `[reuse]` helper lemmas are themselves proved; `hide_lemma` only hides.
  - Restrictions/assumptions embedded in lemmas: e.g. EMV `OneCard()` premise in the `_minimal` lemmas; assumption that "terminals do not complete high-value, contactless transactions with cards that (apparently) do not support cardholder verification" [C05-S5, lines 779-782].
  - Tool-bug workaround: 5G README states lemmas "annotated ... with `attack (stored)` cannot be disproven automatically, due to a bug in the Tamarin-prover", so attacks are stored as interactive proof scripts (`examples/ccs18-5G/README.md`).
  - Unverified translator: m4 macro preprocessing (TLS 1.3), Makefile/m4-style configuration generation (EMVerify). The model-to-specification correspondence is manual in all three studies.
- Arquint et al. TCB: Gobra/VeriFast soundness (Assumption 1), the I/O-spec generator (its output is covered by the metatheorem proof in the paper's appendix, not mechanised in the paper text read), trusted crypto and network stubs ("trusted by assumption and thus also not verified"), unverified setup code, and Assumption 2 [C05-S8, Section 6.4].

## 7. Scale, effort, automation
| Study | Model size | Effort | Run time / resources | Automation |
|---|---|---|---|---|
| TLS 1.3 d21 | 2,129 lines m4 model + 959 lines lemmas (repo); 65 MB of stored proofs | "approximately 3 months worth of work" [C05-S1, line 1031] | "proving the entire model takes about a week of work"; "over 10GB RAM just to load, and can easily consume 100GB RAM"; "an automatically-computed proof was almost 1 million lines long" [C05-S1, lines 1034-1038] | Mixed; bold lemmas in Figure 7 "required manual interaction via the Tamarin visual interface" |
| 5G AKA | ~500 LoC model; 124 lemmas, ~1000 LoC; ~1000 LoC Python oracles [C05-S3] | oracles "represents several person-months" [C05-S3, lines 1022-1023] | "Tamarin needs ca. 5 hours to automatically establish all the proofs and find all the attacks" [C05-S3, line 1055] | Fully automatic with oracles, except stored attacks (tool bug) |
| EMV | 731-846 LoC per target model; 55 (contact) / 60 (contactless) rules; 40 target models [C05-S5, Tables I-II] | "over six months of full-time work" [C05-S5, lines 292-293] | 43 s to 6h03m per model, server with 256 GB RAM, "at most 20GB of RAM per model" | Automatic with oracles |
| Arquint WireGuard | Tamarin ~250 lines rules + 100 lines lemmas; 608 lines verified Go; 3,936 lines spec/annotations (1,241 = 32% generated) | not stated | Tamarin "about 3 minutes"; Gobra ~148 s (initiator) / ~138 s (responder) | "We had to manually write three lemmas and an oracle ... to help the tool terminate" [C05-S8, Section 6.3] |

## 8. Limitations stated by the authors (verbatim)
- Tamarin manual: "Tamarin operates in the symbolic model and thus can only capture attacks within that model, and given a certain equational theory. Currently, apart from the builtins, only subterm-convergent theories are supported. The underlying verification problems are undecidable in general, so Tamarin is not guaranteed to terminate." [C05-S6]
- TLS 1.3: "We model six out of the eight required properties, omitting downgrade protection and the protection of endpoint identities." [C05-S1, line 438]; "we also provide support for PSKs but limit the number of PSKs offered to one per handshake." [line 952]; "Because we model perfect cryptography and cover TLS 1.3 only, we consider these attacks to be out of scope" [lines 959-961]; "We do not currently model using different signing algorithms or their effects on security." [Figure 8 annotation, lines 1327-1328]
- 5G AKA: "Simplifications Made. As usual in the symbolic model, we omit message bit lengths. ... We do not model the optional, non-normative protection against wrapping around the SQN ... We omit this optional mechanism [5G-GUTI]. Authentication tokens do not expire in our model" [C05-S3, lines 979-995]; "it is very challenging, and out of the scope of the present paper, to assess if all currently specified subsequent procedures ... correctly mandate the use of this key confirmation roundtrip" [lines 1309-1314]; "Tamarin's fully automatic mode fails to prove relevant security properties and even extremely weak properties such as the full executability of the protocols." [lines 999-1001]
- EMV: "random number generators are assumed to be sound ... Therefore, attacks of this kind are usually not part of a symbolic analysis that examines the specification (not the implementation) for logical errors. Our analysis thus does not uncover Bond et al.'s attacks" [C05-S5, lines 235-240]; "Of course this is only one part of the overall picture, as attackers can still exploit implementation weaknesses; but it is a substantial part and is also a prerequisite for any 'full stack' effort to formally develop a verified protocol down to the level of code." [lines 133-137]
- Arquint: "the protocol verified is a highly abstract version of the actual protocol executed and there is a priori no formal link between these two versions." [C05-S8, Section 1]; "we removed features not included in our Tamarin model, namely DDoS protection ... we removed metrics and load balancing, which requires complex concurrency reasoning currently not supported by Gobra ... we wrote stubs for cryptographic and network operations and equipped them with trusted specifications" [Section 6.4]; "Our framework is based on a Dolev-Yao attacker ... we currently only support trace properties. ... How to obtain implementation-level equivalence guarantees from a symbolic model is an open problem." [Section 8]

## 9. Independent evidence
- Bugs/flaws found: TLS 1.3 client/server authentication-status mismatch after post-handshake client auth; "The TLS working group has decided not to fix this behaviour for TLS 1.3" [C05-S1, lines 1200-1202]. 5G AKA: KSEAF/SUPI binding weakness "introduced in the version v0.8.0 of the standard (published in March 2018)" [C05-S3, lines 1100-1102, 1325]; traceability attack against active attacker [lines 1385-1393]. EMV: Visa contactless PIN bypass tested on live terminals (~$190 and 182.35 CHF transactions) and offline unauthenticated-transaction attack (not field-tested) [C05-S5]; "On April 30th, 2020, we notified Visa" [line 198].
- Cross-tool comparison by the TLS authors: ProVerif draft-18 models by Bhargavan et al. "did not uncover the potential mismatch between client and server view that we describe" [C05-S1, lines 186-188] (see REF-C06).
- Reuse: Arquint et al. list TLS, 5G, EMV, Noise as prior Tamarin case studies [C05-S8, Section 2.1] and reuse Tamarin for WireGuard code verification; implementation interoperable "with the official implementation and can delegate OS traffic over a VPN".

## 10. Reproducibility
Not run: sandbox disk limit (instruction from /root; no tool installs or runs). Exact commands from the artefacts:
- TLS 1.3: `cd src/rev21 && make && make proofs` (Makefile target `proofs`; INSTALL.md: "make proofs runs the Tamarin prover on all lemmas which can be automatically proven"); paper reports ~100 GB RAM needed.
- 5G AKA: `tamarin-prover --heuristic=O --oraclename=5G_AKA.oracle proofs/5G_aka_attack_noninjectiveagreement_seaf_ue_kseaf_keyConf_noRev.spthy` (examples/ccs18-5G/README.md), Tamarin 1.4.0.
- EMV: `make generic=Contactless kernel=Mastercard auth=CDA CVM=OnlinePIN value=High` (EMVerify README.md), Tamarin 1.5.1.
Stored proof outputs in the repos (`proofs/summaries.txt`, `models-n-proofs/**/*.proof`) were inspected statically.

## 11. Generalisation
- Pattern exemplified: "design-level symbolic model of a standard, verified for unbounded sessions under a Dolev-Yao attacker; attacks found by counterexample traces". It applies to any protocol whose standard can be hand-transcribed into rewrite rules; it does not touch a codebase.
- Reusable sub-patterns: (i) annotated-RFC side-by-side mapping for model coverage transparency (TLS); (ii) systematic minimal-assumption search by disproving weakened lemmas (5G); (iii) configuration-generated target models from one generic model (EMV); (iv) model-to-code trace-inclusion via generated I/O specs (Arquint).
- What applying it to an arbitrary codebase would need (from Arquint): a Tamarin model in their rule format; a separation-logic verifier for the code's language that supports abstract predicates and trace inclusion (Gobra, VeriFast; Nagini mentioned as "could easily be extended"); trusted contracts for crypto/network libraries; proof of format unambiguity; removal or modelling of features absent from the model; proof of memory safety first.
- Case-specific preconditions: XOR support was a prerequisite for 5G ("Tamarin has only recently been extended to handle XOR") [C05-S3, line 141]; heavy custom oracles; very large RAM for TLS.

## 12. Bearing on sub-claims (facts only, no verdict)
| Sub-claim | Fact from this reference | Source ID |
|---|---|---|
| H1 | Tamarin expresses protocol state machines, unbounded sessions, branching/loops, mutable counters (multiset naturals), XOR, DH, compromise oracles and privacy (observational equivalence) as multiset rewriting plus first-order trace formulas. Time and probability are not modelled in these studies. | C05-S1, C05-S3, C05-S5 |
| H2 | Properties proved at scale for full TLS 1.3 d21 (all modes), 5G AKA, 40 EMV configurations; cost: 3-6+ person-months, oracles of ~1000 LoC Python, up to 100 GB RAM, interactive guidance for some TLS lemmas; tool may not terminate. | C05-S1, C05-S3, C05-S5, C05-S6 |
| H3 | The three case studies have no link to implementation code. Arquint et al. give a trace-inclusion link to Go (Gobra) and Java (VeriFast) code with trusted crypto/network stubs; WireGuard needed feature removal. | C05-S5, C05-S8 |
| H4 | The Arquint route is language-agnostic only up to the availability of a separation-logic verifier for the language; the paper names Gobra, VeriFast, Nagini. | C05-S9 | Tamarin manual, chapter 7 "Property Specification" (sections Trace Properties, Secrecy, Authentication, Restrictions) | https://tamarin-prover.com/manual/master/book/007_property-specification.html | P | [SECT] | Defines trace properties as guarded first-order formulas over action facts and timepoints; `K(x)` is the action of the built-in adversary-deduction rule; lemmas hold for all traces unless marked exists-trace. |
| C05-S8 |
| H5 | In these studies, protocol-level proofs exist without any code check; in Arquint, code is checked against a generated spec while security proofs stay at model level ("decouples proving the security properties from verifying the implementation's correctness"). | C05-S8 |
| D | "verify" in these papers = Tamarin proves a trace property for all traces of the model (unbounded); "falsified" = attack trace found. EMV distinguishes "design" from "implementations". Arquint: "verified implementation" = Hoare triple with I/O spec proved in Gobra/VeriFast. | C05-S1, C05-S5, C05-S8 |

## 13. Conflicts
- Line counts: 5G paper says model "roughly 500 LoC" and lemmas "ca. 1000 LoC" with "124 different lemmas" [C05-S3]; the current repo file `5G-AKA-nonBindingChannel/5G_AKA.spthy` has 2,771 lines and 80 `lemma` declarations, oracle 661 lines vs "∼1000 LoC of Python". Possible explanations (comments, counting across variants, later edits) are not verified.
- TLS 1.3 Tamarin vs ProVerif (REF-C06): Cremers et al. state the ProVerif draft-18 analysis "did not uncover the potential mismatch between client and server view" while noting "In concurrent work by Bhargavan et al. [10], a similar issue was uncovered for the 0.5-round trip time (RTT) case." [C05-S1, lines 186-188, 1176-1177]. Both statements are from the same source; recorded without resolution.

## 14. Sources
| ID | Title | URL | P/S | Depth | Summary |
|---|---|---|---|---|---|
| C05-S1 | Cremers et al., A Comprehensive Symbolic Analysis of TLS 1.3 (CCS 2017) | https://tls13tamarin.github.io/TLS13Tamarin/docs/tls13tamarin-draft21.pdf | P | [FULL] | Tamarin model of all TLS 1.3 d21 modes; proves 6 of 8 claimed properties; finds authentication-status mismatch; 3 months, 100 GB RAM. |
| C05-S2 | TLS13Tamarin repository @ 6726237 | https://github.com/tls13tamarin/TLS13Tamarin | P | [CODE] | m4 model (2,129 lines) and lemmas (959 lines) for rev21, plus stored proofs and summaries. |
| C05-S3 | Basin et al., A Formal Analysis of 5G Authentication (CCS 2018, arXiv v4) | https://arxiv.org/abs/1806.10360 | P | [FULL] | Tamarin model of 5G AKA with SQN counters and XOR; minimal-assumption tables; finds binding-channel weakness and traceability attack. |
| C05-S4 | tamarin-prover examples/ccs18-5G @ 9228c58 | https://github.com/tamarin-prover/tamarin-prover/tree/develop/examples/ccs18-5G | P | [CODE] | Model files, oracles, stored attack proofs and analysis_results for 5G AKA. |
| C05-S5 | Basin, Sasse, Toro-Pozo, The EMV Standard: Break, Fix, Verify (S&P 2021, arXiv v3) | https://arxiv.org/abs/2006.08249 | P | [FULL] | Tamarin model of EMV contact/contactless, 40 configurations; Visa PIN bypass demonstrated on real terminals; verified terminal-side fixes. |
| C05-S5b | EMVerify repository @ 5922009 | https://github.com/EMVrace/EMVerify | P | [CODE] | Generic models, oracles, generated target models and stored proofs. |
| C05-S6 | Tamarin manual, Limitations chapter (repo `manual/src/014_limitations.md` @ 9228c58) | https://tamarin-prover.com/manual/master/book/014_limitations.html | P | [FULL] | Symbolic-model-only, subterm-convergent theories, non-termination, approximate observational equivalence. |
| C05-S8 | Arquint et al., Sound Verification of Security Protocols: From Design to Interoperable Implementations (S&P 2023; arXiv 2212.04171 v1) | https://arxiv.org/abs/2212.04171 | P | [SECT] (Sections 1-8 read in full; appendices not read; claims used here are quoted from Sections 3, 4, 5.2 and 8) | Generates separation-logic I/O specs from Tamarin; trace-inclusion soundness; verifies 608 lines of wireguard-go in Gobra with trusted stubs. |

## 15. Unverified leads (excluded from evidence)
- Cremers et al. S&P 2016 (TLS 1.3 draft 10, Tamarin), quoted elsewhere in this repo as "analyse the specification only"; not re-read here.
- Dehnel-Wild and Cremers, 5G AKA draft vulnerability page (Feb 2018), cited as [19] in C05-S3; not fetched.
- Tamarin WireGuard model `examples/wireguard/wireguard.spthy` (seen in the repo listing but not read).
- Noise protocol Tamarin analysis (Girol et al.), cited in C05-S8 as [23]; not read.
- Arquint et al. artefact repository ([20] in C05-S8); not cloned (disk limit).
