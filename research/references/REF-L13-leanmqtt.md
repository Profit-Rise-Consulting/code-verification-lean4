# REF-L13: LeanMqtt (verified MQTT 5.0 packet parser/serializer in Lean 4)

Source depth counts (section 14): 1 [FULL], 1 [SECT], 1 [CODE]. Evidence level: static-inspected (not built). AI provenance: none-found.

## 1. Identity
| Field | Value |
|---|---|
| Subject (protocol or algorithm) | MQTT 5.0 control-packet wire format (fixed header, variable header; payloads not implemented) |
| Spec it targets (e.g. RFC 8446 section) | OASIS MQTT Version 5.0 (2019), Appendix B mandatory conformance statements; syntax-related subset only |
| Organisation / authors | Eduardo Sandalo Porto (UFMG, formerly USP) and Ana Cristina Vieira de Melo (USP). CNPq grant #134250/2025-7 (S1) |
| Primary URLs | https://github.com/edusporto/lean-mqtt ; paper `paper.pdf` in repo (SBLP 2026, 8-11 Sept 2026, São Paulo); Zenodo artifact https://doi.org/10.5281/zenodo.22181982 (cited in S1, not opened) |
| Repo, commit SHA, last commit date, licence | edusporto/lean-mqtt @ `af23e7bea58dace8e361f6d77d69ce6d7254105c`, 2026-09-21T16:51:40-03:00, Apache-2.0 (`LICENSE`) |
| Lean toolchain (or other tool and version) | `leanprover/lean4:v4.34.0`; no Mathlib dependency (manifest has no external Lean packages for the main build; `docbuild/` adds doc-gen4) |
| Maturity (research prototype / production / abandoned / AI-generated) | Research prototype. README: "A work-in-progress formally verified parser and serializer". Single committer in the shallow history (Eduardo Sandalo Porto). No README or commit statement of AI generation; the paper names LLM pipelines only as future work. Not flagged as AI-generated. |
| AI provenance | none-found (no agent configuration files, no AI commit trailers in the shallow history, no author statement of AI use) |

## 2. Claim as stated by the authors (verbatim)
- Abstract (S1): "we present a case study in implementing and formally verifying a parser for the MQTT 5.0 protocol using Lean 4. Our approach includes three components: (I) correct-by-construction modeling of the packet format, leveraging dependent types to preclude invalid packets and encode data-dependent structural variations; (II) a layered, proof-carrying parser library utilizing state monads to process inputs whose exact layout depends on runtime parsed values; and (III) formal verification of parser completeness (roundtrip) and soundness (reconstruction) with respect to a serializer."
- S1 §1: "In this preliminary phase of the project, we have implemented and verified most of components C1, C2, and C3 for the fixed and variable headers of MQTT control packets, leaving payloads as future work."
- README (S2): "A successful run of `lake build` shows that the proofs discussed in the paper are verified by the Lean kernel, thus, the parser is sound and complete."
- S1 §6: "Of the 68 syntax-related statements identified, our implementation formally satisfies 34, covering fixed and variable headers."

## 3. What is actually formalised
The protocol is modelled purely as a byte format. There is no state, no client/broker roles, no message sequencing, no environment and no adversary. `Parser` is a state-transformer over `List UInt8` with `Option` failure.

The central interface, `LeanMqtt/Core/Codec.lean:19-31`:
```lean
class Codec (α : Type) where
  parser : Parser α
  serialize : α → List UInt8

class LawfulCodec (α : Type) [Codec α] where
  roundtrip : ∀ (a : α) {rest : List UInt8},
    Codec.parser.run (Codec.serialize a ++ rest) = some (a, rest)
  reconstruct : ∀ {a : α} {input rest : List UInt8},
    Codec.parser.run input = some (a, rest) → input = Codec.serialize a ++ rest
```
Packet composition, `LeanMqtt/Packets/Basic.lean:21-33,52-60`:
```lean
structure RawPacket where
  fh : FixedHeader
  vh : VarHeader fh
  pl : Payload fh vh
...
abbrev PacketPred (p : RawPacket) : List Condition :=
  [ensure! p.vh.byteSize + p.pl.byteSize = p.fh.remaining_len.val]
abbrev Packet :=
  PredType RawPacket PacketPred
```
The variable header is dependently typed on the fixed header (`VarHeader fh`). The payload is a placeholder, `LeanMqtt/Packets/Payload/Basic.lean:6-11`:
```lean
/-- The Payload of an MQTT control packet is currently unimplemented. -/
structure Payload (fh : FixedHeader) (vh : VarHeader) where
  -- TODO: Implement Payload variations
def Payload.serialize (_ : Payload fh vh) : List UInt8 := []
def Payload.parser : Parser (Payload fh vh) := pure {}
```
Top-level theorems, `LeanMqtt/Packets/Proofs.lean:9,17,50,55`:
```lean
theorem RawPacket.roundtrip (p : RawPacket) {rest : List UInt8} :
    RawPacket.parser.run (p.serialize ++ rest) = some (p, rest)
theorem RawPacket.reconstruct {p : RawPacket} {input rest : List UInt8} :
    RawPacket.parser.run input = some (p, rest) → input = p.serialize ++ rest
theorem Packet.roundtrip (p : Packet) {rest : List UInt8} :
    Packet.parser.run (p.serialize ++ rest) = some (p, rest)
theorem Packet.reconstruct {p : Packet} {input rest : List UInt8} :
    Packet.parser.run input = some (p, rest) → input = p.serialize ++ rest
```
The primitives (`UInt`, `VarInt`, `Str`, `SizedList`, `OptType`, `PredType`, `ConstVal`) each carry roundtrip and reconstruct theorems. Example: `LeanMqtt/Primitives/VarInt/Proofs.lean:10` `VarInt.roundtrip`, `:120` `VarInt.reconstruct`. `VarInt` is `Fin 268_435_456` (`VarInt/Basic.lean:23,29`), and the parser rejects non-minimal encodings (`VarInt/Basic.lean:63-65`, comment "The spec forbids non-minimal encodings (MQTT-1.5.5-1)").

`Main.lean` only prints "If you're seeing this, the theorems were proven succesfully!". README: the binary "currently does not run the parser".

## 3a. Predicate semantics
Headline theorems: `Packet.roundtrip` and `Packet.reconstruct` (`LeanMqtt/Packets/Proofs.lean:50,55`), and their `RawPacket` versions (`:9,17`), quoted in §3.
- `Parser` (`LeanMqtt/Core/Parser/Basic.lean:18`, verbatim): `abbrev Parser (α : Type) := StateT (List UInt8) Option α`. So `p.run input = some (a, rest)` means that running the state transformer on `input` succeeds, returns `a` and leaves the suffix `rest` unconsumed. `none` is the only failure mode (the module doc says `Except` errors are future work, `:15-16`).
- `Packet` (`LeanMqtt/Packets/Basic.lean:52-60`) is `PredType RawPacket PacketPred`, where `PacketPred p = [ensure! p.vh.byteSize + p.pl.byteSize = p.fh.remaining_len.val]`.
- `PredType` (`LeanMqtt/Primitives/PredType/Basic.lean:50-51`, verbatim): `abbrev PredType (α : Type) (gen : α → List Condition) : Type := { val : α // AllHold gen val }`, with `abbrev AllHold ... := ∀ c ∈ gen val, c.prop` (`:36-37`). Its parser (`:61-66`) parses an `α` and then fails unless `AllHold gen val` holds.
- Premises: `roundtrip` has none beyond a value `p` and an arbitrary suffix `rest`. `reconstruct` assumes only that the parse succeeded.
- What is guaranteed: for every value of the Lean `Packet` type, parsing its serialization (followed by any bytes) returns exactly that packet and the unread suffix. Any successful parse consumed exactly the serialization of the returned packet. This makes the parser a left inverse of the serializer and makes the encoding unambiguous.
- What is not guaranteed: that the Lean types or the serializer match the MQTT 5.0 prose (both are the authors' encoding); anything about payloads (`Payload` is an empty stub, so its parser consumes nothing); behaviour on malformed input beyond returning `none`; any client/broker behaviour.

## 4. Property classes covered
| Class | Status | Evidence |
|---|---|---|
| Parsing | partial | Roundtrip and reconstruction (an injectivity/inverse pair against the authors' own serializer) for fixed and variable headers. Payload is an empty stub, so `Packet` covers only payload-less byte strings. 34 of 68 syntax-related conformance statements claimed (S1 §6; list in `conformance.md:5-38`). |
| Functional correctness | partial | Parser equals inverse of serializer. No independent spec of MQTT bytes beyond the Lean types themselves. |
| Safety invariant | no | No state machine. |
| Liveness | no | |
| Refinement | no | |
| Symbolic / computational security | no | |
| Constant-time | no | |
| Other | Correct-by-construction types: packet validity predicates (`PredType`, `ensure!`), e.g. remaining-length consistency. |

## 5. Link to implementation code
Lean is the implementation: the parser is executable Lean code and the theorems are about that code. No deployed MQTT implementation (Mosquitto, EMQX, etc.) is linked. S1 §8 proposes, as future use, that "this verified parser can serve as an executable specification to validate a highly optimized implementation in another language through differential and property-based testing, following a verification-guided development approach". This is not implemented in the repo.

## 6. Trusted computing base and soundness hygiene
Static counts over 42 `.lean` files at `af23e7b` (comments stripped before counting): `sorry` 0, `admit` 0, `axiom` 0, `native_decide` 0, `implemented_by` 0, `extern` 0, `opaque` 0, `partial` 0. TCB is the Lean kernel and compiler plus the trust that the Lean types match the MQTT prose. The paper notes (S1 §5) "formalization uncovered a bug in our original VBI parser: failing to check VBI canonicity invalidates reconstruction". The theorems are relative to the authors' serializer. A serializer and parser that both misread the spec the same way would still satisfy them.

## 7. Scale, effort, automation
- 42 Lean files, 4,751 lines in total (4,720 under `LeanMqtt/`). `Proofs.lean` files plus `Helpers/` = 2,829 lines; other definition files = 2,032 lines. 151 `theorem`/`lemma` declarations.
- S1 §6: "Our preliminary results show a roughly 1:1 ratio of non-proof to proof lines".
- Automation: custom tactics in `Helpers/ParserTactics.lean`, `CrushLits.lean`, `NatifyUInt8.lean`; `omega` for VBI arithmetic (S1 §5). S1: "While this final process is still manual, we plan on fully automating the proofs for each composite type".
- Effort is not reported.

## 8. Limitations stated by the authors (verbatim)
- S1 §1: "leaving payloads as future work."
- S1 §6: "The remaining statements largely include unimplemented payloads or complex edge cases postponed to future work."
- S1 §6: "Much of the protocol's behavior resides solely in the specification's natural language prose. Translating full English specifications into formal models remains a challenge beyond our current scope".
- S1 §2: "Our focus, however, is strictly on MQTT's syntax, which is the packet format".
- S1 §8: "future work will extend this library to payloads, custom error handling, and array parsing."
- README: "we currently only implement the parser, with future work planned on implementing the full MQTT broker/client."

## 9. Independent evidence
The paper appeared at SBLP 2026, a peer-reviewed venue per S1's running header. The SBC Open Library landing page lists the same title and authors (see §15; not used as evidence). One self-reported bug was found during formalisation (VBI canonicity, S1 §5). No third-party adoption, review or criticism was found.

## 10. Reproducibility
Not built: sandbox disk limit; queued for CI reproducibility job. An earlier attempt to install the `v4.34.0` toolchain failed with `No space left on device (os error 28)` while extracting, before any Lean file was compiled. Exact command: `cd lean-mqtt && lake build` (toolchain `leanprover/lean4:v4.34.0`, no Mathlib, cap 25 min). The theorem statements and hygiene counts above come from static inspection only.

## 11. Generalisation
Pattern: a "lawful codec" (parser/serializer pair with roundtrip and reconstruction proofs) built compositionally from verified primitives. Dependent types carry data-dependent layouts (`VarHeader fh`) and validity predicates (`PredType`). This is the same property shape as EverParse/Narcissus (both cited in S1). To apply it to an arbitrary codebase you would need (a) a Lean model of the wire format, (b) proofs per composite type, which the authors aim to automate via an eDSL, and (c) an implementation link to the production parser, for example differential testing against the Lean executable, which this reference does not supply. Preconditions specific to this case: MQTT's simple TLV/VBI framing, payload-free scope, no Mathlib.

## 12. Bearing on sub-claims (facts only, no verdict)
| Sub-claim | Fact from this reference | Source ID |
|---|---|---|
| H1 | MQTT 5.0 fixed/variable header formats are expressed as dependent Lean types with validity predicates; protocol behaviour (state, sessions, QoS flows) is not expressed. | S1, S2 |
| H2 | Roundtrip and reconstruction are kernel-checked by the authors' account for all header primitives and the `Packet` type, with 0 sorry/axiom; the theorems were not re-checked here. | S1, S2 |
| H3 | Lean code is the implementation; no link to a mainstream-language MQTT implementation. Differential testing is proposed but not done. | S1 §8, S2 |
| H4 | The approach is specific to wire formats; generalisation is proposed via a future eDSL. | S1 §8 |
| H5 | Parser correctness is proved without any protocol-level (behavioural) proof. | S1 |
| D | "formally verifying a parser" = proving roundtrip (completeness) and reconstruction (soundness) relative to a serializer; "conformance" = manual classification of spec statements, 34/68 syntax statements "formally satisf[ied]". | S1 §5-6 |

## 13. Conflicts
- README says "the parser is sound and complete". S1 says payloads are unimplemented, and the code's `Payload` is an empty stub (`Payload/Basic.lean:6-11`). So "complete" holds for the header-only `Packet` type, not for MQTT packets with payloads (CONNECT, PUBLISH, SUBSCRIBE). The two statements differ in scope; not resolved here.
- S1 §6 cites "251 mandatory conformance statements". `conformance.md` contains 285 occurrences of `MQTT-` identifiers, because the implemented list at the top repeats IDs that also appear in the body. The two counts measure different things and have not been reconciled.

## 14. Sources
| ID | Title | URL | P/S | Depth | Summary |
|---|---|---|---|---|---|
| S1 | Porto and de Melo, "Foundations for Formally Verified Network Protocol Parsers in Lean 4: A Case Study with MQTT", SBLP 2026 (paper.pdf in repo), accessed 2026-11-07 | https://github.com/edusporto/lean-mqtt/blob/main/paper.pdf | P | [FULL] | Short paper describing the dependent-type packet model, parser combinator library and roundtrip/reconstruction proofs for MQTT 5.0 headers. 34/68 syntax conformance statements; payloads are future work. |
| S2 | edusporto/lean-mqtt source @ af23e7b, accessed 2026-11-07 | https://github.com/edusporto/lean-mqtt | P | [CODE] | 4,751 Lean lines, 151 theorems, 0 sorry/axiom; `LawfulCodec` class with `RawPacket`/`Packet` roundtrip and reconstruct; payload stub. |
| S3 | lean-mqtt `conformance.md` @ af23e7b, accessed 2026-11-07 | https://github.com/edusporto/lean-mqtt/blob/main/conformance.md | P | [SECT] | Sections read: header list of implemented statements (`conformance.md:5-38`) and the syntax/semantics/transport classification. Quote: "Before the list, we consider the following syntax-related statements currently implemented:". Authors' classification of MQTT 5.0 Appendix B statements into syntax, semantics and transport, plus the list of 34 implemented syntax statements. |

## 15. Unverified leads (excluded from evidence)
- SBC Open Library landing page for the paper: https://sol.sbc.org.br/index.php/sblp/article/view/46205. Only page metadata and the abstract were read (fetch truncated).
- Zenodo artifact https://doi.org/10.5281/zenodo.22181982 (not opened).
