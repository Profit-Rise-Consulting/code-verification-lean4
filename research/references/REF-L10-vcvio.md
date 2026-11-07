# REF-L10: VCVio (plus Hopscotch)

## 1. Identity
| Field | Value |
|---|---|
| Subject (protocol or algorithm) | General framework for game-based computational cryptography proofs in Lean 4 (oracle computations as a free monad, probabilistic semantics, relational/unary program logic), with case studies: forking lemma, Fiat-Shamir, Schnorr EUF-CMA, random-oracle commitments, SLH-DSA SUF-CMA in the ROM, ML-DSA/ML-KEM/Falcon specifications. Companion: Hopscotch, a game-hopping (state-separation style) framework built on VCVio's `OracleSpec` API. |
| Spec it targets (e.g. RFC 8446 section) | Textbook game-based security definitions (EUF-CMA, SUF-CMA, IND-CPA, IND-CCA, PRF/PRG); NIST FIPS 203 (ML-KEM), FIPS 204 (ML-DSA), FIPS 205 (SLH-DSA), Falcon. Hopscotch: definitions from Rosulek, "The Joy of Cryptography". |
| Organisation / authors | VCVio: GitHub org `Verified-zkEVM`; paper authors Devon Tuma, Nicholas Hopper (University of Minnesota), Quang Dao, James Waters (Carnegie Mellon University), Alexander Hicks (Ethereum Foundation). Hopscotch: Stefan Dziembowski, Grzegorz Fabiański, Daniele Micciancio, Rafał Stefański (arXiv affiliations include University of Warsaw and IDEAS Research Institute; per-author affiliations not recorded here). |
| Primary URLs | https://github.com/Verified-zkEVM/VCVio ; https://eprint.iacr.org/2026/899 ; https://eprint.iacr.org/2024/1819 ; https://github.com/ravst/Hopscotch ; https://arxiv.org/abs/2608.06261 |
| Repo, commit SHA, last commit date, licence | VCVio `0b69833ce5cac9a7ba0349803a5b7b5e4a365127`, 2026-11-06T22:19:36Z, Apache-2.0. Hopscotch `2699b4feda99047cd009021fd6f14e046a022da8`, 2026-08-10T11:54:55+02:00, MIT. |
| Lean toolchain (or other tool and version) | VCVio `leanprover/lean4:v4.35.0` with Mathlib, cslib, PolyFun, QuantumFoundations. Hopscotch `leanprover/lean4:v4.28.0`, Mathlib `8f9d9cff`, VCVio `e7f4c8d`. |
| Maturity (research prototype / production / abandoned / AI-generated) | VCVio: active research library (738 commits since 2024-03-11, 12+ contributors), heavily AI-assisted with human-authored statements and review (see §9). Hopscotch: "an active, experimental research project", AI-assisted (Codex, DeepSeek, Aristotle). |

## 2. Claim as stated by the authors (verbatim)
VCVio `README.md`:
> "VCVio is a Lean library for machine-checked proofs about cryptographic games, primitives, protocols, and implementations."

> "SLH-DSA has a proved quantitative SUF-CMA bound (strong unforgeability: no valid message-signature pair the signer never returned, which gives EUF-CMA at the same bound) in the classical random-oracle model with three oracles (the tweakable hash, `Hmsg` and `PRFmsg`), per public seed and averaged over seed distributions (`securityTarget_two_one`)."

ePrint 2026/899 abstract:
> "We present VCVio, a foundational Lean 4 framework that closes both gaps using established ideas from programming-language theory: algebraic effects and handlers for oracles, and a modular relational program logic for tactics. Concretely, a computation with oracle access is the free monad over the polynomial functor determined by its oracle specification … rewinding reduces to deterministic transcript replay without internal adversary state."

ePrint 2026/899 §10:
> "To our knowledge, this enables the first foundational mechanization of a general forking lemma and the unforgeability of Schnorr signatures, avoiding the rewindability axioms and explicit state management required by the recent EasyCrypt formalization [FJ25] of Firsov and Janků."

Hopscotch `README.md`:
> "HOPSCOTCH is a framework for formalizing game-hopping proofs in Lean. … The project currently builds without `sorry` declarations in active code."

## 3. What is actually formalised
Model: `OracleComp spec α` is the free monad over the polynomial functor of an `OracleSpec`; `ProbComp` is the special case with uniform-sampling oracles. Semantics are a measure-valued `evalDist` (denotational) and `simulateQ` (handler-based operational). Random oracles, logging, caching and reprogramming are handlers; the forking lemma uses replay of a recorded transcript. Adversaries are arbitrary Lean terms of type `OracleComp …`; query bounds are explicit hypotheses; "efficiency predicate `isPPT` is left abstract; users specialize it" (ePrint 2026/899, Appendix). Security statements are concrete (explicit advantage inequalities in `ENNReal`).

Core definition, `VCVio/OracleComp/OracleComp.lean` (lines ~23-30 of the file at `0b69833`):
```lean
@[reducible]
def OracleComp {ι : Type u} (spec : OracleSpec.{u, v} ι) :
    Type w → Type (max u v w) :=
  PFunctor.FreeM spec.toPFunctor
```

Schnorr EUF-CMA, `VCVio/Schemes/Schnorr/Signature.lean:202`:
```lean
theorem signature_euf_cma [Fintype F] (g : G)
    (hg : Function.Bijective (· • g : F → G))
    (M : Type) [DecidableEq M]
    (adv : SignatureAlg.UnforgeableAdversary (signature F G g M))
    (qS qH : ℕ)
    (hQ : ∀ pk, FiatShamir.signHashQueryBound (M := M) (Commit := G) (Chal := F)
      (S' := G × F) (oa := adv.main pk) qS qH) :
    let eps :=
      SignatureAlg.unforgeableAdvantage (FiatShamir.runtime (Commit := G) (Chal := F) M) adv -
      ((qS : ENNReal) * (qS + qH) * ((Fintype.card F : ℝ≥0∞)⁻¹))
    eps * (eps / (qH + 1 : ENNReal) - FiatShamir.challengeSpaceInv F) ≤
      Pr{let x ← dlogExperiment g (dlogReduction F G g M adv qH)}[x = true]
```

SLH-DSA, `HashSig/SLHDSA/Security/CoverageBound.lean:350`:
```lean
theorem securityTarget_two_one (laws : core.ByteLaws) (hd : core.KeyDiscipline vp)
    (hcard : Nat.card core.Y ≤ Nat.card core.SkPrf) (e : core.SkSeed ≃ core.Y)
    (optRand : PublicKeyCore core → ProbComp core.Y) :
    SecurityTarget core e optRand 2 1
```
Note: `laws : core.ByteLaws` and `hd : core.KeyDiscipline vp` are hypotheses on the byte-level primitives.

ML-KEM: only interfaces. `LatticeCrypto/MLKEM/Security.lean` docstring: "A complete IND-CCA theorem additionally requires a concrete K-PKE correctness bound, a fixed decisional MLWE problem, explicit adversary reductions, and a Fujisaki–Okamoto bound whose error terms use nonnegative types. Those ingredients are not consequences of the interface packaging provided here."

Hopscotch main soundness theorem, `Hopscotch/ComputationalIndistinguishability/Soundness.lean:312`:
```lean
theorem computationalSoundness {Idx : Type} {Assumptions : IndAssumptions Idx}
      {q_b : ENat}
      {I : Type} {O : OracleSpec I} {o₁ o₂ : OracleImpl O}
      (ind : IndistinguishableI Assumptions q_b o₁ o₂) :
      advBoundQ Assumptions q_b O o₁ o₂ (assumptionCountLower (assumptionCountingFin ind))
```
Hopscotch case studies (`Hopscotch/Examples/Proofs/`): `secureGGM_of_securePRG` (`GGMSecurePRF.lean:567`), `indCpaAndMacUfImpliesIndCcaEncryptThenMacFam` (`EncryptThenMacIndCca.lean:181`), `ddhImpliesElGamalOTUCPubFam` (`DDHImpliesElGamalOTUCPub.lean:206`), IND-RAND ⇒ IND-CPA, one-time secrecy ⇒ IND-CPA (public key), length-tripling PRG. These are `noncomputable def`s producing `IndistinguishableI` proof objects, converted to advantage bounds by `computationalSoundness`.

## 4. Property classes covered
| Class | Status | Evidence |
|---|---|---|
| Functional correctness | partial | Scheme correctness lemmas (e.g. `Falcon.verify_sign_correct`, `LatticeCrypto/Falcon/Security.lean:91`), NTT certificate lemmas, FIPS 205 component conformance against pinned ACVP vectors (README). "A complete FIPS conformance result is not proved" (README). |
| Safety invariant | no | Not the framework's subject. |
| Liveness | no | — |
| Refinement | partial | `shakeRomScheme_map_shakeAnswer` relates the ROM scheme to FIPS 205 SHAKE internals (README); Interop hax/Aeneas bridges are experimental and dormant. |
| Symbolic security | no | (ePrint 2026/899 cites a separate Lean formalisation of computationally sound symbolic crypto [DFMS25] using the same oracle API.) |
| Computational security | yes | Schnorr EUF-CMA to DLog, forking lemma, Fiat-Shamir, SLH-DSA SUF-CMA in ROM; Hopscotch GGM, EtM IND-CCA, ElGamal from DDH. PPT not formalised. |
| Parsing | partial | SLH-DSA canonical public-key/signature codecs (`wireStrongAdvantage_le_securityBound_validatedParams`); ML-KEM byte encoding lemmas. |
| Constant-time | no | — |

## 5. Link to implementation code
Mostly spec-level: theorems are about Lean definitions of schemes, not deployed code. Some schemes have executable Lean concrete implementations (`LatticeCrypto/*/Concrete`), tested against native C implementations through `Extern/` FFI (`@[extern]` bindings, "used in tests"). `Interop/` contains bridges to Lean emitted by Rust frontends hax and Aeneas, but `Interop/README.md`: "Lean 4.31 baseline: Interop is dormant. Neither backend is resolved by Lake, and CI intentionally excludes the `Interop` aggregate target." The 2026 paper: "We also plan to link Lean specifications to efficient executable code, connecting design-level security with implementation-level correctness." Hopscotch: spec-level only.

## 6. Trusted computing base and soundness hygiene
VCVio counts (rg over `*.lean`, excluding `third_party/`; tokens in comments/docstrings filtered by hand):
- `sorry`: 0 in proof code. The HEAD commit is "chore(LatticeCrypto): remove the last four sorry placeholders (#1043)". `scripts/axiom_baseline.json` records `"sorry": []` and `"nonstandard": []`, maintained by `scripts/AxiomSweep.lean` ("whole-library kernel-level axiom and `sorry` accounting").
- `axiom` declarations: 7, all in `scripts/VCVioAxiomSweepTestFixtures/Tainted/*` (deliberately tainted fixtures for the sweep checker). None in libraries.
- `native_decide`: 1 real use, `Interop/Hax/CenteredBinomial.lean:144` (dormant Interop lib, outside CI). Interop also uses `bv_decide`.
- `@[extern]`: 37 (Extern/Falcon 7, Extern/Hashing 5, Extern/MLDSA 10, Extern/MLKEM 10, plus `LatticeCrypto/MLDSA/Concrete/LawBounds.lean`, `lakefile.lean`, `scripts/InitSweep.lean`). `Extern/MLDSA/Laws.lean` states the concrete-instance laws "reach the native FFI surface and lives here rather than in `LatticeCrypto/`".
- `implemented_by`: 16 (NTT kernels in `LatticeCrypto/*/Concrete/NTT.lean`, `LatticeCrypto/Ring/Kernel.lean`, `Extern/MLDSA/*`).
- `partial def`: 19 (tactic internals, widgets, scripts, Falcon FFT/sampler).
- Interop TCB contract: "Pulling hax / aeneas into the build adds a sizeable trusted computing base (extraction frontend, Charon, the backend's own runtime axioms) … nothing in core VCVio depends on it", enforced by `scripts/check-interop-isolation.sh` in CI.
- Modelling TCB: definitions of security games, `isPPT` left abstract, byte-law hypotheses (`ByteLaws`, `KeyDiscipline`) in SLH-DSA.

Hopscotch: 0 `sorry` in code (6 occurrences, all in comments), 0 `axiom`, 0 `native_decide`, 0 `extern`; `ARISTOTLE_SUMMARY.md` records `#print axioms` for `secureGGM_of_securePRG` as `[propext, Classical.choice, Quot.sound]` (self-reported by the AI tool run).

## 7. Scale, effort, automation
- VCVio: 1,612 `.lean` files / 437,855 lines; by directory `VCVio` 193,251, `HashSig` 97,819, `VCVioTest` 37,745, `Examples` 26,105, `HashSigTest` 22,952, `LatticeCrypto` 19,005, `Extern` 14,177, `ToMathlib` 11,597, `VCVioWidgets` 2,879, `LatticeCryptoTest` 2,944, `Interop` 1,756. ~14,687 `theorem`/`lemma` declarations.
- ePrint 2026/899 Table 4 ("Case-study formalization size, measured in non-comment Lean lines", columns Defs./Thms., Proofs, Total): Random-oracle commitments 1,113 / 3,371 / 4,484; Forking lemma 860 / 3,685 / 4,545; Fiat–Shamir and Schnorr 3,112 / 5,273 / 8,385; Total 5,085 / 12,329 / 17,414.
- Automation: custom `vcgen`/`rvcgen` tactics on a relational extension of Loom (POPL 2026); interactive game-hop widgets.
- Hopscotch: 70 files / 12,827 lines (`ComputationalIndistinguishability` 4,596, `Examples` 3,974, `Tactic` 2,712, `ObservationalEq` 621, `Comp` 604, `Indistinguishability` 296); 265 theorem/lemma declarations; `game_hopping` tactic; 251 commits since 2026-02-18.

## 8. Limitations stated by the authors (verbatim)
- VCVio README: "Variable-length XOF game transport, SLH-DSA-specific honest-work certificates, byte-level range certificates, and a second construction remain open. The SHA-2 parameter sets have no single-oracle model. A complete FIPS conformance result is not proved".
- `LatticeCrypto/MLKEM/Security.lean`: "A complete IND-CCA theorem additionally requires a concrete K-PKE correctness bound, a fixed decisional MLWE problem, explicit adversary reductions, and a Fujisaki–Okamoto bound … Those ingredients are not consequences of the interface packaging provided here."
- ePrint 2026/899 Appendix: "The efficiency predicate isPPT is left abstract; users specialize it".
- ePrint 2026/899 §9 (AI): "Agents given a large, unstructured objective (for instance, "complete the EUF-CMA proof of Schnorr signatures") drift: they produce low-quality intermediate lemmas, spend tokens on irrelevant refactors, or declare success on proofs whose sorrys have been pushed deeper into the call graph."
- `Interop/README.md`: "Hax does not yet support Lean 4.31; Aeneas does, but its VCVio bridge has not been revalidated."
- Hopscotch (arXiv 2608.06261 §4): "formalizing probabilistic polynomial time would remove the remaining externally checked complexity conditions and yield fully mechanized computational-security statements." The paper states Hopscotch does not verify polynomial-time complexity of reductions and relies on concrete bounds (paraphrase of §2.4 via LLM extraction; see S6).

## 9. Independent evidence
- Adoption: ePrint 2026/899: "VCVio is already being adopted as a base layer for more complex interactive proofs in the ArkLib library"; Hopscotch depends on VCVio's oracle API; the paper notes "Dziembowski et al.'s concurrent Lean formalization of computationally sound symbolic cryptography [DFMS25]" uses the same API.
- No independent audit found.
- AI-generation / assistance (Conventions rule 7):
  - VCVio commit history (tree-less clone, 738 commits): 225 commit messages carry an AI co-author trailer or session link (Claude Fable 5/5.1, Claude Opus 4.8/5/5.5, Cursor, Codex), and 16 carry `Co-authored-by: Aristotle (Harmonic)`. ePrint 2026/899 §9: "A substantial fraction of the VCVio codebase has been touched by LLM coding agents at some point during development, and external automated proof-search systems have opened pull requests against our repository." and "humans state theorems, agents propose proofs, the compiler checks."
  - Hopscotch README: "The project is being developed with assistance from the LLM-based tools Codex from OpenAI and Aristotle from Harmonic." arXiv 2608.06261 "Use of AI tools": "For the Lean code, we used Codex with GPT-5.5 (18), DeepSeek-V4 (10) and Aristotle by Harmonic (2). … Aristotle was used in a more autonomous mode: given the statement that every observationally equivalent pair of oracles factors through two abstractions, it constructed a proof together with several intermediate definitions and lemmas." `ARISTOTLE_SUMMARY.md` records Aristotle filling `rState2Rstate_non_dist`, described as "the hard one (the original authors marked it "quite hard" and left it open)".

## 10. Reproducibility
Not built: sandbox disk limit; queued for CI reproducibility job. (VCVio is a Mathlib-dependent 438k-line build; no attempt made after the coordinator's disk instruction.) Exact commands:
```
git clone --depth 1 https://github.com/Verified-zkEVM/VCVio && cd VCVio && git checkout 0b69833ce5cac9a7ba0349803a5b7b5e4a365127
lake exe cache get && lake build
./scripts/validate.sh --axioms          # axiom/sorry sweep against scripts/axiom_baseline.json
git clone --depth 1 https://github.com/ravst/Hopscotch && cd Hopscotch && git checkout 2699b4feda99047cd009021fd6f14e046a022da8
lake exe cache get && lake build
```

## 11. Generalisation
Pattern: foundational (kernel-checked, no extra axioms) game-based cryptographic proofs, with oracle access as a free monad and oracle behaviour as composable handlers; reductions and game hops are Lean terms with explicit concrete advantage bounds. Hopscotch layers a proof-object calculus (`IndistinguishableI`) plus a soundness theorem converting it to an advantage bound.
For arbitrary codebases: VCVio proves properties of Lean models of schemes, not of existing implementations. Linking to code requires a translation step (hax/Aeneas from Rust, both outside CI) or a separate refinement proof. Preconditions: the scheme is specified as a probabilistic program with oracle access; query bounds are stated; efficiency (PPT) is accepted as an informal/external side condition.

## 12. Bearing on sub-claims (facts only, no verdict)
| Sub-claim | Fact from this reference | Source ID |
|---|---|---|
| H1 | Lean 4 with Mathlib can express oracle-access probabilistic programs, random oracles, rewinding (via transcript replay) and concrete advantage bounds; PPT is left as an abstract predicate. | S1, S2, S4 |
| H2 | Kernel-checked EUF-CMA for Schnorr from DLog, a general forking lemma, and SLH-DSA SUF-CMA in a 3-oracle ROM exist with 0 `sorry` and no library axioms; ML-KEM IND-CCA is not proved. | S1, S2, S3 |
| H3 | Proofs are about Lean models; implementation linkage via hax/Aeneas is experimental and excluded from CI; native code appears only through FFI used in tests. | S3, S5 |
| H4 | Applies to cryptographic schemes expressible as `OracleComp`; does not directly verify existing code. | S2, S3 |
| H5 | The framework proves computational security of designs, separately from implementation correctness; the paper names linking them as future work. | S2 |
| D | Authors use "machine-checked proofs", "foundational", "formally verified" for kernel-checked Lean results; "proved quantitative SUF-CMA bound" with explicit model qualifiers. | S1, S2 |
| AI | Both projects report heavy LLM assistance with human-stated theorems; one failure mode named is agents "declar[ing] success on proofs whose sorrys have been pushed deeper". | S2, S6, S8 |

## 13. Conflicts
- ePrint 2026/899 §10 lists ML-KEM/ML-DSA/Falcon as "future work", while the current repo contains `LatticeCrypto/` specifications and concrete implementations for all three. Consistent with the paper predating the commit; recorded only. The repo still does not claim ML-KEM IND-CCA.
- Hopscotch README pins VCVio commit `e7f4c8d` and Lean v4.28.0, whereas VCVio HEAD is on v4.35.0; Hopscotch therefore depends on an older VCVio. Not a contradiction, but results are not re-checked against current VCVio.

## 14. Sources
| ID | Title | URL | P/S | Depth | Summary |
|---|---|---|---|---|---|
| S1 | VCVio README.md @ 0b69833 (accessed 2026-11-07) | https://github.com/Verified-zkEVM/VCVio | P | FULL | Framework scope, SLH-DSA claim and open items, repo layout, build. |
| S2 | Tuma, Dao, Waters, Hicks, Hopper, "VCVio: Verified Cryptography in Lean via Oracle Effects and Handlers", ePrint 2026/899 (accessed 2026-11-07 via eprint.iacr.asia mirror PDF) | https://eprint.iacr.org/2026/899 | P | FULL | Design, case studies with LOC table, AI-assistance section, future work. |
| S3 | VCVio source code @ 0b69833 (accessed 2026-11-07) | https://github.com/Verified-zkEVM/VCVio | P | CODE | Theorem statements, TCB counts, axiom baseline, Interop/README, ML-KEM Security docstring. |
| S4 | Tuma, Hopper, "VCVio: A Formally Verified Forking Lemma and Fiat-Shamir Transform, via a Flexible and Expressive Oracle Representation", ePrint 2024/1819 (accessed 2026-11-07) | https://eprint.iacr.org/2024/1819 | P | FULL | Original framework paper; comparison with EasyCrypt, SSProve, CryptHOL, FCF. |
| S5 | VCVio commit history (tree-less clone, accessed 2026-11-07) | https://github.com/Verified-zkEVM/VCVio/commits/main | P | CODE | 738 commits; AI co-author trailer counts. |
| S6 | Dziembowski, Fabiański, Micciancio, Stefański, "Game Hopping in Lean", arXiv 2608.06261v1 (accessed 2026-11-07) | https://arxiv.org/abs/2608.06261 | P | PARTIAL | First 60k characters read in full (abstract, intro, AI-tools note, framework, start of case studies); conclusion and limitations obtained via LLM extraction of the full HTML. |
| S7 | Hopscotch README.md and source @ 2699b4f (accessed 2026-11-07) | https://github.com/ravst/Hopscotch | P | CODE | Status, AI assistance statement, soundness theorem, case-study proofs, counts. |
| S8 | Hopscotch ARISTOTLE_SUMMARY.md @ 2699b4f (accessed 2026-11-07) | https://github.com/ravst/Hopscotch/blob/main/ARISTOTLE_SUMMARY.md | P | FULL | AI tool run logs: which sorries Aristotle filled and the reported axiom closure. |

## 15. Unverified leads (excluded from evidence)
- CatCrypt (Lean, SSProve-derived, hax extraction) mentioned in ePrint 2026/899 §9; "its repository is not publicly available at the time of writing".
- [DFMS25] Dziembowski et al., Lean computationally sound symbolic cryptography using VCVio's oracle API; not located.
- ArkLib adoption of VCVio (claimed in S2); not inspected.
