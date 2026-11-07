# REF-L15: NyxFoundation libp2p-fv and formal-eth

Both repositories turn out to contain no Lean code. `libp2p-fv` is a research proposal: a property inventory of obligations, with a tool-selection document that assigns Lean 4 one role. `formal-eth` is a curated "awesome list" of Ethereum formal-verification projects. This dossier records exactly what each contains so that neither is cited as verification evidence.

## 1. Identity
| Field | Value |
|---|---|
| Subject (protocol or algorithm) | libp2p-fv: safety obligations for Ethereum GossipSub over QUIC v1 + TLS 1.3 libp2p identity binding, multistream-select, Protobuf RPC, Snappy/SSZ and gossip validation. formal-eth: index of Ethereum FV projects (cryptography, protocol, application, tools, papers). |
| Spec it targets (e.g. RFC 8446 section) | libp2p-fv: libp2p specs (TLS handshake, multistream-select, GossipSub v1.x), RFC 9000/9001 (QUIC), RFC 8446, Ethereum consensus p2p-interface. It cites "exact source sections" per obligation, e.g. Q-01.01 → "Packet protection". |
| Organisation / authors | Nyx Foundation. The last commit in both repos is by `shouki <shoukitsuda@icloud.com>`. |
| Primary URLs | https://github.com/NyxFoundation/libp2p-fv ; https://github.com/NyxFoundation/formal-eth |
| Repo, commit SHA, last commit date, licence | libp2p-fv @ `1f25f348524db6556c7e1fd942f37a4ad5ea4f5d`, 2026-11-05T15:03:01+09:00, no LICENSE file. formal-eth @ `33f5f0f85a1ad7ca3620fc3eb46b31824adf0d62`, 2026-10-29T21:56:31+09:00, MIT. |
| Lean toolchain (or other tool and version) | None in either repo (no `lean-toolchain`, no `.lean` files). libp2p-fv proposes Quint, TLC, Apalache, Lean 4 and Tamarin. |
| Maturity (research prototype / production / abandoned / AI-generated) | libp2p-fv: research proposal (pre-prototype), Markdown only. formal-eth: curated list, maintained (last commit "Merge pull request #6 from NyxFoundation/update-2026-10"). No AI-generation statement found in either. |

## 2. Claim as stated by the authors (verbatim)
- libp2p-fv README: "This project aims to specify the safety of Ethereum GossipSub over QUIC, from TLS 1.3 certificate-to-Peer-ID binding through Snappy/SSZ processing to the checks that gate gossip acceptance. Its intended contribution is a hierarchy of independently checkable obligations connecting authenticated transport to validated message delivery."
- libp2p-fv README: "This is a research proposal, not a verified implementation or a completed end-to-end theorem; intended novelty requires further literature review."
- libp2p-fv `inventory.md`: "All obligations are **proposed**, not proved or model-checked. The original 61 candidates are tracked as **53 active parent families and 8 excluded families**. Active parents have 160 child obligations in the four linked documents."
- libp2p-fv `docs/verification-tool-selection.md`: "Use **Quint** for executable state-machine specifications, **TLC** and **Apalache** for model checking, **Lean 4** for deductive proofs and contract composition, and **Tamarin** for symbolic security protocol analysis."
- formal-eth README: "A curated list of awesome Ethereum formal verification tools, projects, and resources".

## 3. What is actually formalised
Nothing is mechanised in either repository.
- libp2p-fv: 7 Markdown files, 1,136 lines (`README.md` 49, `inventory.md` 184, `docs/verification-tool-selection.md` 95, and four `docs/properties/*.md` totalling 808). Each obligation has the fields "ID | Premise | Event | Required conclusion | Basis | Depends on" and is tagged S (specification-derived), D (design obligation) or K (conditional cryptographic claim). Example, `docs/properties/transport-and-authentication.md` Q-02.01: "Local required peer checks have not succeeded | Application dispatch attempts to read buffered bytes | No event labels those bytes as belonging to an authenticated libp2p peer. | D [TLS peer authentication]". Scope is a layer chain written as text (`inventory.md` "Scope" block): "Untrusted candidate address / expected Peer ID -> UDP / QUIC v1 + TLS 1.3 libp2p identity binding -> QUIC stream + multistream-select -> GossipSub RPC / Protobuf / routing and scoring -> Snappy block / SSZ / topic and fork context -> Ethereum gossip validation -> validated delivery or payload-forwarding authorization".
- formal-eth: a README table of about 45 rows across Projects (Cryptography, Protocol, Application), Specifications, Tools, Reports and Research. No row concerns networking, libp2p or p2p transport; grep found no "libp2p", "gossip", "p2p" or "network" entries. Lean-relevant rows include ArkLib, leanSig/leanSPHINCS XMSS (SUF-CMA in the ROM), Poseidon2/OpenVM/Pico chip soundness, etheorem consensus specs, formal-leanSpec, and Verity.

## 4. Property classes covered
| Class | Status | Evidence |
|---|---|---|
| Functional correctness | no (proposed only) | libp2p-fv F-, C-, V- families (encoding/validation), proposed |
| Safety invariant | no (proposed only) | G-, M-, R- families, proposed |
| Liveness | no; explicitly excluded | inventory "Excluded: availability, delivery success/probability/deadlines, mesh recovery ..." |
| Refinement | no | E-07.02 "Implementation correspondence ... is out of scope, not established" (tool-selection doc) |
| Symbolic / computational security | no (proposed: Tamarin for symbolic, K-claims for computational) | Q-01, A- families |
| Parsing | no (proposed) | F- and C- families |
| Constant-time | no; side channels excluded | inventory "Excluded: ... side channels" |
| Other | formal-eth: literature index only | |

## 5. Link to implementation code
None. libp2p-fv tool selection: "It excludes ... verification of a particular implementation" and "no implementation has been selected" (`inventory.md`).

## 6. Trusted computing base and soundness hygiene
Not applicable: there are zero Lean files, so the sorry/axiom/native_decide counts are all 0 vacuously. libp2p-fv states its own evidence discipline: "A finite TLC result, bounded Apalache check, symbolic Tamarin result, inherited result, and Lean theorem must remain distinct in reports. Do not convert one into another by naming the same property."

## 7. Scale, effort, automation
libp2p-fv: 1,136 Markdown lines; 53 active parent families, 8 excluded, 160 child obligations. formal-eth: one README of about 45 table rows. No effort data.

## 8. Limitations stated by the authors (verbatim)
- libp2p-fv README: "Availability, dissemination guarantees, DoS resistance, global memory/CPU bounds, Req/Resp, discovery internals, TCP fallback, full block validation, fork choice, and execution validity are excluded."
- Tool selection: "All of these are proposed activities. No model checking, proof, or cross-tool correspondence is claimed by this document."
- Tool selection, Lean row: "A theorem establishes only its formal statement and assumptions. It does not automatically establish correspondence with a Quint or Tamarin model."
- Tool selection: "Quint, TLC, Apalache, Lean, and Tamarin do not share a theorem automatically."

## 9. Independent evidence
None for either repo; no external citation or review was found. The prior work that libp2p-fv lists is ACL2s GossipSub (Kumar et al., ACL2 2023; IEEE S&P 2024), Floodsub refinement (ACL2 2025) and QUIC-Ivy (Crochet et al., 2021). This prior work is real-looking but was not opened here (see §15).

## 10. Reproducibility
Nothing to build: no Lean or other mechanised artefact exists in either repo at the pinned commits. Not built for that reason (the sandbox disk limit was also in force).

## 11. Generalisation
libp2p-fv contributes a methodology rather than a result: an obligation inventory (premise/event/conclusion/basis/dependencies, each tagged S/D/K) written before tools are chosen, with explicit tool roles. In that split, Lean handles data/encoding contracts and cross-layer composition, Quint/TLC/Apalache handle interleavings, and Tamarin handles symbolic authentication. Results across tools must be kept distinct. Precondition for reuse: a fixed protocol profile and pinned spec revisions, which the authors say are not yet chosen ("Fix a profile and source commits before reporting any verification result").

## 12. Bearing on sub-claims (facts only, no verdict)
| Sub-claim | Fact from this reference | Source ID |
|---|---|---|
| H1 | A networking group plans to use Lean 4 only for "selected general invariants, data and arithmetic properties, and composition theorems". State-machine interleavings go to Quint/TLC/Apalache and symbolic authentication to Tamarin. | S1 |
| H2 | No Lean proof exists in libp2p-fv or formal-eth. | S1, S2 |
| H3 | Implementation correspondence is out of scope in libp2p-fv. | S1 |
| H4 | libp2p-fv considers Lean insufficient alone for the network stack and proposes a multi-tool approach. | S1 |
| H5 | No fact: nothing has been verified, so no claim about a verified parser or protocol can be drawn. | S1 |
| D | libp2p-fv separates S (spec-derived), D (design) and K (conditional crypto) obligations, and separates finite/bounded/symbolic/deductive evidence. | S1 |

## 13. Conflicts
Prompt and earlier notes vs. sources: the earlier research files list these repositories among Lean network verification work. At the pinned commits, neither contains Lean code. formal-eth has no networking entries. libp2p-fv describes itself as "a research proposal".

## 14. Sources
| ID | Title | URL | P/S | Depth | Summary |
|---|---|---|---|---|---|
| S1 | NyxFoundation/libp2p-fv, all 7 Markdown files @ 1f25f34, accessed 2026-11-07 | https://github.com/NyxFoundation/libp2p-fv | P | FULL | Proposal and property inventory (53 parents, 160 children) for Ethereum GossipSub over QUIC; tool selection Quint/TLC/Apalache/Lean/Tamarin; no proofs. |
| S2 | NyxFoundation/formal-eth README @ 33f5f0f, accessed 2026-11-07 | https://github.com/NyxFoundation/formal-eth | P | FULL | Awesome-list of Ethereum FV projects; Lean entries are zk/crypto/consensus; no networking entries. |

## 15. Unverified leads (excluded from evidence)
- Kumar et al., Verification of GossipSub in ACL2s, https://arxiv.org/abs/2311.08859 ; artifacts https://github.com/gossipsubfm/gossipsubfm
- Kumar and Manolios, Floodsub correctness, https://arxiv.org/abs/2507.19013
- Crochet et al., Verifying QUIC implementations using Ivy, https://arxiv.org/abs/2503.01374
- NyxFoundation/formal-leanSpec and NyxFoundation/verity (linked from formal-eth; Lean consensus work, not networking).
