# REF-L12: Clean (Verified-zkEVM / zkSecurity circuit verification)

## 1. Identity
| Field | Value |
|---|---|
| Subject (protocol or algorithm) | Embedded Lean 4 DSL and verification framework for zero-knowledge arithmetic circuits (AIR, PLONK, R1CS): gadgets carry soundness and completeness proofs against a `Spec`; channel-based composition of multi-table (multi-AIR) ensembles toward zkVMs. Case studies include Keccak-f permutation, SHA-256, BLAKE3, Poseidon (circomlib port), circomlib gadgets, Fibonacci/opcode VMs, FemtoCairo. |
| Spec it targets (e.g. RFC 8446 section) | Per-gadget Lean `Spec` predicates, e.g. Keccak spec `out_state.value = keccakPermutation state.value`; reference specs in `Clean/Specs/` (SHA256, BLAKE3, Poseidon, Keccak). Circomlib port targets iden3 circomlib semantics. |
| Organisation / authors | zkSecurity (licence "Copyright (c) 2024-2025 zkSecurity, LLC"), hosted under GitHub org `Verified-zkEVM`. Top commit authors: Gregor Mitscha-Baude (incl. "Gregor's Bot"), Yoichi Hirai, Giorgio Dell'Immagine. |
| Primary URLs | https://github.com/Verified-zkEVM/clean ; https://clean.zksecurity.xyz ; https://blog.zksecurity.xyz/posts/clean-verified-zkvms/ ; https://blog.zksecurity.xyz/posts/poseidon-clean/ |
| Repo, commit SHA, last commit date, licence | `f8a4194c604488cb732024890aebd083968222ab`, 2026-11-05T18:59:53+01:00, MIT (the `Clean/Circomlib/` folder is LGPL, per its README, because circomlib is LGPL). |
| Lean toolchain (or other tool and version) | `leanprover/lean4:v4.35.0`, Mathlib, CompPoly. |
| Maturity (research prototype / production / abandoned / AI-generated) | Active industrial research framework (5,168 commits since 2024-07-30; 504 since 2026-08-01); heavily AI-assisted (see §9). |

## 2. Claim as stated by the authors (verbatim)
Project site (clean.zksecurity.xyz):
> "Clean is a framework for writing formally verified zero-knowledge circuits created by zkSecurity for the community of zk builders. It enables proving the equivalence between low-level, usually optimized circuits and their high-level desired behavior."

> "the circuit implementation, both now and when you change it in 6 months, is guaranteed to behave exactly as you described."

> "Clean decouples circuits from their arithmetization. One definition, one proof, compiled to the constraint system you need."

Blog, "Clean: From Verified Circuits to Verified zkVMs" (2026-06-05):
> "*global* soundness must make no mention of extra assumptions like `BytesChannel.Guarantees`. Instead, it can only assume that all channels in the ensemble are balanced, which is what the proof system enforces."

Blog, "Verifying Poseidon in Clean" (2026-05-04):
> "**Soundness.** If an assignment satisfies all constraints of the Clean circuit, then the output is `poseidon1Opt input`. **Completeness.** For every input, the honest computation produces a witness satisfying all constraints."

## 3. What is actually formalised
Model: a circuit is a Lean monadic program (`Circuit F`) producing a list of operations (witness allocations, equality constraints, lookups, channel pushes/pulls, subcircuits). `ConstraintsHold` interprets operations over an `Environment` (witness assignment). Correctness is two properties against user `Assumptions`/`Spec`.

Soundness, `Clean/Circuit/Formal.lean:198`:
```lean
def Soundness (F : Type) [FiniteField F] (main : Var Input F → Circuit F (Var Output F))
    [elaborated : ElaboratedCircuit F Input Output main]
    (Assumptions : Input F → Prop) (Spec : Input F → Output F → Prop) :=
  -- for all environments that determine witness assignments
  ∀ offset : ℕ, ∀ env : Environment F,
  -- for all inputs that satisfy the assumptions
  ∀ input_var : Var Input F, ∀ input : Input F, eval env input_var = input →
  Assumptions input →
  -- if the constraints hold
  ConstraintsHold.Soundness env (main input_var |>.operations offset) →
  -- the spec holds on the input a…
```
Bundled gadget, `Clean/Circuit/Formal.lean:239`:
```lean
structure FormalCircuit (F : Type) [FiniteField F] (Input Output : TypeMap) [ProvableType Input] [ProvableType Output]
    extends base : FormalCircuitBase F Input Output where
  Assumptions (_ : Input F) : Prop := True
  Spec : Input F → Output F → Prop
  soundness : Soundness F base.main Assumptions Spec
  completeness : Completeness F base.main Assumptions
```
`DeterministicFormalCircuit` adds `uniqueness` (spec determines output) — relevant to under-constrained-circuit bugs.

Example: Keccak-f permutation gadget, `Clean/Gadgets/Keccak/Permutation.lean` — `Spec` includes `out_state.value = keccakPermutation state.value`; `theorem soundness : Soundness (F p) main Assumptions Spec` (line ~39).

Ensemble level (blog and `Clean/Air/FlatEnsemble.lean`): `Statement ens publicInput := ∃ witness, witness.publicInput = publicInput ∧ witness.Constraints ∧ witness.BalancedChannels`, and `Soundness ens Assumptions Spec := ∀ publicInput, Assumptions publicInput → ens.Statement publicInput → Spec publicInput`; library lemmas e.g. `tableSoundness_of_soundChannels` (`Clean/Air/FlatEnsemble.lean:626`).

What is not modelled: the polynomial IOP / proof system itself (channel balance is assumed to be "what the proof system enforces"), the cryptographic soundness of the SNARK/STARK, the prover/verifier code, and the backend translation to R1CS/AIR/PLONK.

## 4. Property classes covered
| Class | Status | Evidence |
|---|---|---|
| Functional correctness | yes (circuit level) | Every `FormalCircuit` proves constraints ⇒ `Spec` (soundness) and honest witness ⇒ constraints (completeness); Keccak, SHA-256, BLAKE3, Poseidon gadgets. |
| Safety invariant | partial | VM state-channel reasoning proves "there exists a valid trace from the public initial state to the public final state" (blog). |
| Liveness | no | — |
| Refinement | yes (circuit ↔ spec) | Site: "proving the equivalence between low-level, usually optimized circuits and their high-level desired behavior"; `DeterministicFormalCircuit.uniqueness`. |
| Symbolic security | no | — |
| Computational security | no | Proof-system soundness and zero-knowledge are outside scope. |
| Parsing | no | (Poseidon blog: "the proof does not mechanically parse `poseidon.circom`".) |
| Constant-time | no | — |

## 5. Link to implementation code
Lean is the circuit implementation: the gadget written in Lean is the object proved. Deployment requires compiling to a backend constraint system: `Clean/Backends/` (Circom/R1CS export with WASM witness compile, Plonky3 printing; 3,705 lines). Static inspection found 1 `theorem` in `Clean/Backends/`; the backend compilers are not verified and are validated by tests (e.g. `Clean/Backends/Circom/TestWasmCompile.lean`). Circomlib ports are hand models: Poseidon blog: "The Clean circuit is a direct hand model of the same optimized arity-1 structure. The correspondence is small and explicit, but it is not a verified compiler pipeline from Circom source to R1CS." Guarantee holds for the Lean circuit model; the exported constraint system and any external prover are outside the proof.

## 6. Trusted computing base and soundness hygiene
Counts (rg over 213 `.lean` files, comments filtered):
- `sorry`: 10, all in `Clean/Utils/Test/TestCircuitProofStart.lean` (a tactic test file). 0 in gadgets/framework.
- `axiom` declarations: 0. `@[extern]`: 0. `implemented_by`: 0. `partial def`: 16.
- `native_decide`: 111 raw occurrences. Concentrated in `Clean/Specs/SHA256.lean` (41), `Clean/Specs/BLAKE3/ChunkProcessingTests.lean` (18), `Clean/Specs/PoseidonOptimized.lean` (17), `Clean/Utils/Primes.lean` (10), `Clean/Specs/Poseidon.lean` (7), `Clean/Specs/BLAKE3.lean` (4), circomlib `CompConstant*` (5), witness-generation tests. `Clean/Utils/Primes.lean` uses `native_decide` for primality of small/medium field primes, so field instances for those primes depend on `Lean.ofReduceBool` (compiler-in-TCB). BN254 primality is now `⟨BN254.ScalarField_is_prime⟩` "Proven prime via CompPoly's Pratt certificate" (`Clean/Specs/Poseidon.lean:26-27`).
- Modelling TCB: user `Spec`/`Assumptions`; channel balance assumed to be enforced by the proof system; the backend exporters; hand modelling of circomlib sources.
- Blog: "since the `Channel.Guarantees` are not mentioned in global soundness, they are untrusted code … Auditors don't have to worry about them (just about the global `Assumptions` and `Spec`)."

## 7. Scale, effort, automation
- 213 `.lean` files / 54,500 lines; ~1,568 `theorem`/`lemma` declarations. By directory: `Circuit` 12,516, `Gadgets` 9,294, `Utils` 7,417, `Air` 5,606, `Circomlib` 5,317, `Examples` 3,832, `Backends` 3,705, `Specs` 3,301, `Table` 1,425, `Tables` 1,083, `Types` 721.
- Effort: 5,168 commits over ~27 months. Poseidon blog: "After weeks of work, the only remaining `sorry` was a primality proof for the BN254 scalar field".
- Automation: `circuit_proof_start`, `circuit_proof_all`, `circuit_norm` simp set, `elaborate_circuit_with`; many simple gadgets close with a one-line tactic (blog `isZero` example).

## 8. Limitations stated by the authors (verbatim)
- Verified-zkVMs blog: "Proving the local-to-global lift for completeness in Clean is future work."
- Same: "We do not prove that every state interaction in the trace has an independent external meaning. We prove the VM statement that matters: there exists a valid trace from the public initial state to the public final state."
- Same: "The ensemble is sound, but meaningless since the soundness theorem has an unsatisfiable assumption. … Without completeness, it is too easy to prove a theorem whose assumptions are impossible."
- Poseidon blog: "the proof does not mechanically parse `poseidon.circom`. … it is not a verified compiler pipeline from Circom source to R1CS." and "the equality `poseidon1Opt = poseidon1` is not yet proved for all field elements. This is a useful validation layer, not a formal equivalence proof." "Currently, the proof is about arity 1 Poseidon."
- `Clean/Gadgets/Keccak/Permutation.lean`: "-- TODO why does this time out?? -- completeness" (completeness discharged via a separately proved lemma).

## 9. Independent evidence
- zkSecurity is itself an audit firm; the blog index lists "Checking the Checkers and Auditing the Ironwood FV" (2026-10-05, three-week audit of a formalization) and a Zero Knowledge podcast episode on Clean and zkGolf (2026-10-21) — titles only (see §15).
- No independent third-party audit of Clean located.
- AI assistance (Conventions rule 7): commit history (tree-less clone) shows 571 `Co-Authored-By: Claude` trailers, 542 "Generated with [Claude" lines, plus Claude Fable 5 (107), Opus 4.6 (81), Opus 4.7 (34), Sonnet 4.5 (33), Opus 4.8 (21), Opus 5 (15), Copilot (13); bot authors "Gregor's Bot" (448) and "Gregor's Bots" (112). The project site states: "LLMs allow "scaling engineering effort", because they generate implementations *and* proofs" and "the part of the code that requires review is the specification". The repo contains `CLAUDE.md` agent instructions.

## 10. Reproducibility
Not built: sandbox disk limit; queued for CI reproducibility job. Exact commands:
```
git clone --depth 1 https://github.com/Verified-zkEVM/clean && cd clean && git checkout f8a4194c604488cb732024890aebd083968222ab
lake exe cache get && lake build
```

## 11. Generalisation
Pattern: write the constraint system as a Lean program, bundle each gadget with `Spec`, soundness and completeness (optionally uniqueness), compose by specification through subcircuits, and lift local proofs to whole-ensemble soundness via channel abstractions with acyclic "finished channel" ordering. Under-constrained bugs show up as unprovable soundness/uniqueness.
To apply to an arbitrary circuit codebase (Circom, Halo2, Plonky3 AIR): the existing circuit must be re-expressed or hand-modelled in Clean (no verified parser/import); the generated constraints must be trusted to match deployed constraints, or compared via the backend exporter and tests. Preconditions: finite-field arithmetic circuits; specs expressible as Lean predicates; the proof system enforces channel balance.

## 12. Bearing on sub-claims (facts only, no verdict)
| Sub-claim | Fact from this reference | Source ID |
|---|---|---|
| H1 | Lean 4 expresses circuits as monadic programs with constraint semantics, soundness/completeness/uniqueness definitions and multi-table channel balance. | S1, S2 |
| H2 | 54.5k lines with 0 framework/gadget `sorry`, 0 axioms; Keccak, SHA-256, BLAKE3, Poseidon gadgets proved; 111 `native_decide` uses widen the TCB to the Lean compiler for some specs/primes. | S1, S3 |
| H3 | Proofs are about the Lean circuit; export to R1CS/AIR is unverified (1 theorem in Backends); circomlib ports are hand models. | S1, S3 |
| H4 | Applies to arithmetic-circuit code once re-expressed in Clean; not to arbitrary software. | S1, S2 |
| H5 | Circuit correctness is proved separately from proof-system cryptographic soundness, which is assumed via "channels are balanced". | S2 |
| D | Authors use "formally verified", "Perfect by Construction", "guaranteed to behave exactly as you described", while blogs state explicit trust boundaries. | S2, S3, S4 |
| AI | Over a thousand commits carry Claude/Copilot trailers; authors position LLMs as generating implementations and proofs with humans reviewing specs. | S2, S5 |

## 13. Conflicts
- `Clean/Circomlib/Poseidon.md:162` states "`instance : Fact (Nat.Prime BN254_PRIME)` is declared with `by sorry`", and the Poseidon blog (2026-05-04) calls primality "the only remaining `sorry`". At `f8a4194`, `Clean/Specs/Poseidon.lean:27` instead proves it via CompPoly's Pratt certificate, and no such `sorry` remains. Resolved in favour of the code at the stated commit; the `.md` note is stale.
- Site claim "guaranteed to behave exactly as you described" versus blog trust boundaries (hand-modelled Circom, unverified export, unproved `poseidon1Opt = poseidon1`). Recorded, not resolved.

## 14. Sources
| ID | Title | URL | P/S | Depth | Summary |
|---|---|---|---|---|---|
| S1 | Clean source code @ f8a4194 (accessed 2026-11-07) | https://github.com/Verified-zkEVM/clean | P | CODE | Definitions of Soundness/FormalCircuit, gadgets, Backends, counts, Poseidon primality. |
| S2 | Mitscha-Baude, "Clean: From Verified Circuits to Verified zkVMs", ZK/SEC Quarterly, 2026-06-05 (accessed 2026-11-07) | https://blog.zksecurity.xyz/posts/clean-verified-zkvms/ | P | FULL | Channels, ensemble soundness, VM state channels, completeness caveat. |
| S3 | Ochoa, "Verifying Poseidon in Clean: Why the Last 'sorry' Is About Primality", 2026-05-04 (accessed 2026-11-07) | https://blog.zksecurity.xyz/posts/poseidon-clean/ | P | FULL | Poseidon arity-1 soundness/completeness, explicit trust boundaries. |
| S4 | Clean project site (accessed 2026-11-07) | https://clean.zksecurity.xyz | P | FULL | Positioning claims, backends, LLM "spec compilation" vision. |
| S5 | Clean commit history (tree-less clone, accessed 2026-11-07) | https://github.com/Verified-zkEVM/clean/commits/main | P | CODE | 5,168 commits; AI co-author trailer counts. |

## 15. Unverified leads (excluded from evidence)
- zkSecurity, "Checking the Checkers and Auditing the Ironwood FV" (2026-10-05) — title/teaser only; relevant to auditing Lean formalizations.
- Zero Knowledge podcast Episode 414 "zkGolf, AI, Clean and Optimising ZK Circuits" (2026-10-21) — teaser only.
- Original "Introducing clean" blog post (part 1 of the series) — not read.
- Other Verified-zkEVM projects (ArkLib, CompPoly, zkLean) — not inspected in this dossier.
