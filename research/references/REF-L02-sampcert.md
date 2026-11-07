# REF-L02: SampCert (verified differential privacy and discrete samplers)

## 1. Identity
| Field | Value |
|---|---|
| Subject (protocol or algorithm) | Randomised algorithms for differential privacy (DP): discrete Laplace and discrete Gaussian samplers (Canonne, Kamath, Steinke algorithms), an abstract DP framework (pure DP, zCDP, approximate DP), DP mechanisms (noised count/sum/mean, histogram, sparse vector / AboveThreshold), all in a shallow-embedded probability monad `SLang`. Algorithm reference, not a network protocol. |
| Spec it targets (e.g. RFC 8446 section) | Mathematical definitions: discrete Gaussian/Laplace distributions (arXiv 2004.00010), zCDP (Bun and Steinke, arXiv 1605.02065), sparse vector technique (Dwork and Roth) [C1][S1]. No RFC. |
| Organisation / authors | Amazon Web Services / Lean FRO repo. Paper: Markus de Medeiros, Muhammad Naveed, Tancrède Lepoint, Temesghen Kahsai, Tristan Ravitch, Stefan Zetzsche, Anjali Joshi, Joseph Tassarotti, Aws Albarghouthi, Jean-Baptiste Tristan [S1]. "The principal developer of SampCert is Jean-Baptiste Tristan" [C1]. |
| Primary URLs | https://github.com/leanprover/SampCert ; https://arxiv.org/abs/2412.01671 ; DOI https://doi.org/10.1145/3729294 (PACMPL 9, PLDI 2025, Article 191) |
| Repo, commit SHA, last commit date, licence | leanprover/SampCert @ `a2c53bb5ff790e729e252958060699b7eff898b9`, 2026-09-25 13:48:58 -0400, Apache-2.0 [C1] |
| Lean toolchain (or other tool and version) | `leanprover/lean4:v4.34.1`; requires Mathlib @ `v4.34.1` and doc-gen4 [C1]. Deployment path also uses Dafny (SLang → Dafny → Python) [S1]. |
| Maturity (research prototype / production / abandoned / AI-generated) | Production (samplers): "SampCert is deployed and used in the AWS Clean Rooms Differential Privacy service" [C1]; AWS docs: "AWS Clean Rooms Differential Privacy uses the SampCert sampler, a proven correct sampler implementation developed by AWS" [S2]. DP query library is research-grade: "we have not yet deployed them" [S1]. Named authors, peer-reviewed artifact. |
| AI provenance | `none-found`. Basis: full commit history (blob:none clone, 669 commits) has 4 `Co-authored-by` trailers, all naming humans; no AGENTS.md, CLAUDE.md, .claude, .cursor or Copilot config at a2c53bb [C1]. |

## 2. Claim as stated by the authors (verbatim)
- "we present SampCert, the first comprehensive, mechanized foundation for executable implementations of differential privacy. SampCert is written in Lean with over 12,000 lines of proof. It offers a generic and extensible notion of DP, a framework for constructing and composing DP mechanisms, and formally verified implementations of Laplace and Gaussian sampling algorithms." [S1]
- "Prior work has focused on verifying DP at a high level, assuming either that the foundations are correct or that a perfect source of random noise is available." [S1]
- "We have deployed SampCert's verified sampling algorithms at Amazon Web Services." [S1]
- "Our simple probabilistic language enables straightforward extraction of Lean code while minimizing the trusted computing base." [S1]
- README: "A verified implementation using Lean and Mathlib of randomized algorithms including the discrete Gaussian sampler for differential privacy, key results in zero concentrated differential privacy, and some verified (unbounded) private queries." [C1]

## 3. What is actually formalised
Model: programs are mass functions. `SampCert/SLang.lean:28`:
```lean
abbrev SLang.{u} (α : Type u) : Type u := α → ℝ≥0∞
```
Primitive operators carry `@[extern]` attributes so that the same Lean definition has a mathematical meaning (mass function) for proofs and a C++ sampling implementation for execution: `SLang.lean:65 @[extern "prob_Pure"]`, `:71 @[extern "prob_Bind"]`, `:83-84`
```lean
@[extern "prob_UniformByte"]
def probUniformByte : SLang UInt8 := (fun _ => 1 / UInt8.size)
```
and `:143-144 @[extern "prob_While"] def probWhile (cond : T → Bool) (body : T → SLang T) (init : T) : SLang T :=` (loops defined as a limit of truncations `probWhileCut`, `:126`) [C1]. Mechanisms: "abbrev Mechanism (T U : Type) := List T → PMF U" (paper Listing 1) [S1]; databases are lists.

Key theorems, verbatim, commit `a2c53bb` [C1]:

`SampCert/Samplers/Gaussian/Properties.lean:238`
```lean
theorem DiscreteGaussianSample_apply (num : PNat) (den : PNat) (mix : ℕ) (x : ℤ) :
  (DiscreteGaussianSample num den mix) x =
  ENNReal.ofReal (discrete_gaussian ((num : ℝ) / (den : ℝ)) 0 x)
```
`SampCert/Samplers/Gaussian/Properties.lean:361`
```lean
theorem DiscreteGaussianSample_normalizes (num : PNat) (den : PNat) (mix : ℕ) :
  ∑' x : ℤ, (DiscreteGaussianSample num den mix) x = 1
```
`SampCert/Samplers/Laplace/Properties.lean:345`
```lean
theorem DiscreteLaplaceSample_apply (num den : PNat) (x : ℤ) :
  (DiscreteLaplaceSample num den) x = ENNReal.ofReal (((exp (1/((num : NNReal) / (den : NNReal))) - 1) / (exp (1/((num : NNReal) / (den : NNReal))) + 1)) * (exp (- (abs x / ((num : NNReal) / (den : NNReal))))))
```
`SampCert/DifferentialPrivacy/Pure/Mechanism/Properties.lean:126`
```lean
theorem privNoisedQueryPure_DP (query : List T → ℤ) (Δ ε₁ ε₂ : ℕ+) (ε : NNReal) (HN : laplace_pureDP_noise_priv ε₁ ε₂ ε) (bounded_sensitivity : sensitivity query Δ) :
    PureDP (privNoisedQueryPure query Δ ε₁ ε₂) ε
```
`SampCert/DifferentialPrivacy/ZeroConcentrated/Mechanism/Properties.lean:132`
```lean
theorem privNoisedQuery_zCDP (query : List T → ℤ) (Δ ε₁ ε₂ : ℕ+) (bounded_sensitivity : sensitivity query Δ) :
  zCDP (privNoisedQuery query Δ ε₁ ε₂) ((1/2) * ((ε₁ : NNReal) / ε₂) ^ 2)
```
`SampCert/DifferentialPrivacy/Queries/Histogram/Properties.lean:76`
```lean
lemma privNoisedHistogram_DP (HN_bin : dpn.noise_priv ε₁ (ε₂ * numBins) (ε / numBins)) :
    dps.prop (privNoisedHistogram numBins B ε₁ ε₂) ε
```

### 3a. Predicate semantics: what the quoted theorems do and do not guarantee
Verbatim definitions, commit `a2c53bb` [C1]:

`SampCert/DifferentialPrivacy/Neighbours.lean:21`
```lean
inductive Neighbour (l₁ l₂ : List T) : Prop where
  | Addition : l₁ = a ++ b → l₂ = a ++ [n] ++ b → Neighbour l₁ l₂
  | Deletion : l₁ = a ++ [n] ++ b → l₂ = a ++ b → Neighbour l₁ l₂
```
`SampCert/DifferentialPrivacy/Sensitivity.lean:26`
```lean
noncomputable def sensitivity (q : List T → ℤ) (Δ : ℕ) : Prop :=
  ∀ l₁ l₂ : List T, Neighbour l₁ l₂ → Int.natAbs (q l₁ - q l₂) ≤ Δ
```
`SampCert/DifferentialPrivacy/Pure/DP.lean:20,24`
```lean
def DP (m : Mechanism T U) (ε : ℝ) : Prop :=
  ∀ l₁ l₂ : List T, Neighbour l₁ l₂ → ∀ S : Set U,
  (∑' x : U, if x ∈ S then m l₁ x else 0) / (∑' x : U, if x ∈ S then m l₂ x else 0) ≤ ENNReal.ofReal (Real.exp ε)
def PureDP (m : Mechanism T U) (ε : NNReal) : Prop :=
  DP m ε
```
`SampCert/DifferentialPrivacy/ZeroConcentrated/DP.lean:46,53,58`
```lean
def zCDPBound (q : List T → PMF U) (ε : ℝ) : Prop :=
  ∀ α : ℝ, 1 < α → ∀ l₁ l₂ : List T, Neighbour l₁ l₂ →
  RenyiDivergence (q l₁) (q l₂) α ≤ ENNReal.ofReal (ε * α)
def ACNeighbour (p : List T -> PMF  U) : Prop := ∀ l₁ l₂, Neighbour l₁ l₂ -> AbsCts (p l₁) (p l₂)
def zCDP (q : List T → PMF U) (ε : NNReal) : Prop := ACNeighbour q ∧ zCDPBound q ε
```
`SampCert/Util/Gaussian/DiscreteGaussian.lean:29,68`
```lean
def gauss_term_ℝ (σ μ : ℝ) (x : ℝ) : ℝ :=
  Real.exp ((-(x - μ)^2) / (2 * σ^2))
def discrete_gaussian (σ μ : ℝ) (x : ℝ) : ℝ :=
  gauss_term_ℝ σ μ x / ∑' x : ℤ, gauss_term_ℝ σ μ x
```
Noise conditions: `laplace_pureDP_noise_priv (ε₁ ε₂ : ℕ+) (ε : NNReal) : Prop := (ε₁ : NNReal) / ε₂ = ε` (`Pure/Mechanism/Properties.lean:121`); `gaussian_zCDP_noise_priv (ε₁ ε₂ : ℕ+) (ε : NNReal) : Prop := (1/2) * ((ε₁ : NNReal) / ε₂)^2 ≤ ε` (`ZeroConcentrated/System.lean:51`). Mechanism: `privNoisedQuery query Δ ε₁ ε₂ l := DiscreteGaussianGenPMF (Δ * ε₂) ε₁ (query l)` (`ZeroConcentrated/Mechanism/Code.lean:21`), and `DiscreteGaussianGenSample` fixes `mix = 7` (`Samplers/GaussianGen/Code.lean:23`).

What the theorems guarantee and what they do not:
- Neighbouring means adding or deleting one record anywhere in a list ("unbounded" DP). Substituting one record is not a primitive neighbour step, and the paper notes group privacy cannot be expressed because databases are lists [S1]. Privacy claims are relative to this relation only.
- `sensitivity` is defined only for integer-valued queries and is a hypothesis (`bounded_sensitivity`) of each mechanism theorem. The user must prove it for their own query; SampCert supplies such proofs for its own queries, e.g. `exactBinCount_sensitivity` (`Queries/Histogram/Properties.lean:29`), `exactBoundedSum_sensitivity` (`Queries/BoundedSum/Properties.lean:26`), `exactDiffSum_sens` (`Queries/UnboundedMax/Privacy.lean:21`); AboveThreshold takes `∀ i, sensitivity (qs i) 1` as a hypothesis.
- `privNoisedQuery_zCDP` gives ρ-zCDP with ρ = ½(ε₁/ε₂)² when Gaussian noise of scale σ = Δ·ε₂/ε₁ is added. Here `zCDP` also includes absolute continuity between neighbouring outputs. The guarantee covers the probability distribution `PMF ℤ` defined in Lean. It assumes the Lean mass-function semantics of the SLang primitives (`probUniformByte` uniform, `probWhile` as a limit) and says nothing about the executed C++/Python code beyond that model.
- `PureDP` is stated as an ENNReal ratio over all output sets. In ENNReal, `x / 0 = ⊤` for `x ≠ 0` and `0 / 0 = 0`, so this matches the usual multiplicative ε-DP condition (noted from Mathlib ENNReal semantics, not re-proved here).
- `DiscreteGaussianSample_apply` concerns the scale parameter σ = num/den, with `discrete_gaussian` normalised over ℤ, so σ is not a variance. The result holds for every `mix`, which affects only the algorithm (performance), not the distribution. The theorem says nothing about running time. Together with `DiscreteGaussianSample_normalizes` (total mass 1), it implies almost-sure termination within the model but gives no bound on loop iterations.
- None of the theorems covers timing side channels or floating-point implementations (section 8).

The paper-to-code map is in `paper_mapping.md` [C1]. Proofs use Mathlib results such as the Poisson summation formula [C1]. The "adversary" in DP is implicit in the definition (neighbouring databases, quantification over all outputs); there is no protocol, messaging or state.

## 4. Property classes covered
| Class | Status | Evidence |
|---|---|---|
| Functional correctness | proved (distributional) | Samplers' mass functions equal the target discrete Gaussian/Laplace closed forms and normalise to 1 [C1]. |
| Safety invariant | n/a | Not applicable. |
| Liveness | partial | Almost-sure termination of loops; for AboveThreshold an explicit termination assumption `has_lucky` ("This assumption is only used to prove almost-sure termination") [S1]. |
| Refinement | partial | Equalities between program variants, e.g. "the combined equality proofs establish that sv9_aboveThesh = sv1_aboveThresh" [S1]; two Laplace samplers proved correct to allow switching at runtime [S1]. |
| Symbolic security | no | |
| Computational security | partial | Information-theoretic privacy guarantees (pure DP, zCDP) are proved; not cryptographic security. Randomness source assumed uniform (`/dev/urandom`) [S1]. |
| Parsing | n/a | No parser. |
| Constant-time | no | "In the future, we would like to extend SampCert to model and prove non-existence of timing side-channels." [S1] |

## 5. Link to implementation code
Two mechanisms, both "Lean is the implementation" with trusted compilation steps [S1]:
1. Lean FFI: "compilation using the Lean foreign function interface (FFI), allowing us to compile and execute our DP mechanisms while adding only 57 lines of C++ to our trusted computing base." Each SLang primitive is bound to a C++ function (`ffi.cpp`, 69 lines at this commit); "Random sampling occurs in the external function for probUniformByte, which reads and returns a single byte from /dev/urandom" and "The external function for probWhile executes a C++ while loop." [S1][C1]
2. Extraction: "a translation from SLang to Dafny [32] and then from Dafny to Python, which we used in order to deploy our verified sampling algorithms at Amazon Web Services." The translator is a Lean metaprogram: "we use a Lean metaprogram to inspect the syntax tree of a SLang term and translate a limited subset of Lean directly" (code: `SampCert/Extractor/*.lean`, 822 LOC, with `partial def toDafnyExpr` in `Translate.lean:54`) [S1][C1].

Guarantee: the theorems are about the mass-function semantics of the Lean definitions; the executable artefact is correct only if the extern C++ implementations (or the Dafny translation and Dafny→Python compiler) faithfully realise `probPure`/`probBind`/`probUniformByte`/`probWhile`, and the byte source is uniform. "In both cases, we rely on some trusted computing base, either the FFI implementation or the Dafny extraction, and all the toolchains needed to compile their respective output." [S1] Only the samplers are deployed; the deployed service code that calls them (AWS Clean Rooms) is outside the proof [S1][S2].

## 6. Trusted computing base and soundness hygiene
Counts over `.lean` files excluding `.lake`, non-comment lines [C1]:

| Token | Count | Note |
|---|---|---|
| sorry | 0 | |
| admit | 0 | |
| axiom | 0 | (Mathlib's standard classical axioms apply.) |
| native_decide | 0 | |
| implemented_by | 0 | |
| extern | 10 textual; 6 real `@[extern]` attributes | `SLang.lean:65,71,83,143,225` (`prob_Pure`, `prob_Bind`, `prob_UniformByte`, `prob_While`, `my_run`) and `SampCert.lean:63` (`dirty_io_get`); a 7th (`prob_Until`, `SLang.lean:150`) is commented out; remaining hits are Dafny `{:extern}` strings printed by `Extractor/Print.lean`. |
| opaque | 2 | `SLang.lean:226 opaque run (c : SLang.SPMF T) : IO T`; `SampCert.lean:64 opaque DirtyIOGet`. |
| partial | 7 textual; 6 `partial def` | All in `SampCert/Extractor/` (IR, IRToDafny, Translate) — the unverified translator. |

Trusted: Lean kernel + Mathlib; the `@[extern]` C++ (`ffi.cpp`); `/dev/urandom` uniformity; Lean compiler; for the deployed path, the unverified Lean→Dafny metaprogram, the Dafny compiler to Python, and the Python runtime [S1][C1]. `@[extern]` on a `def` with a noncomputable body means the kernel never relates the C++ to the Lean body; this is by design ("Using an external annotation, a programmer can specify that any term be compiled to an arbitrary C++ function, enabling the Lean compiler to treat it as if it were computable") [S1].

## 7. Scale, effort, automation
- Lean LOC at `a2c53bb`: 18,463 in 108 files; 560 `theorem`/`lemma` declarations [C1]. By directory: DifferentialPrivacy 9,760; Samplers 4,085; Util 2,039; Foundations 978; Extractor 822 [C1]. Paper: "over 12,000 lines of proof" [S1]; zCDP instance "around 3000 lines of code" [S1]; private histogram privacy "in fewer than 50 lines of proof" given the generic framework [S1].
- Implementation (trusted C++): 57 lines per paper [S1]; `ffi.cpp` is 69 lines at this commit [C1].
- Person-effort: not reported in the paper sections read.
- Performance: "the code we automatically extract from Lean outperforms the implementation of a discrete Gaussian sampler implemented by Canonne et al." (Python benchmarks) [S1].
- Automation: interactive tactics over Mathlib; no SMT.
- Build time: not built (section 10).

## 8. Limitations stated by the authors (verbatim)
- "Countable Domains. All mechanisms and mass functions in SampCert take place in discrete probability spaces over countable types." [S1]
- "Support for Floating-Point Algorithms. ... since library support for reasoning about floating-point numbers in Lean is currently limited ... obtaining foundational privacy guarantees about floating-point samplers in SampCert would be a significant challenge." [S1]
- "Database Types. Our DP queries, such as the private histogram, represent a database as plain lists, instead of a more realistic type. This means that SampCert cannot express some properties such as group privacy." [S1]
- "Single-Parameter DP definitions. Abstract DP only supports variants of differential privacy with one real parameter" [S1]
- "While SampCert's differentially private query implementations are also executable, we have not yet deployed them, in part because integrating these queries directly into existing, large-scale applications and libraries for differential privacy would be more challenging." [S1]
- "Unlike the FFI, the extraction approach does not support arbitrary computable Lean terms" [S1]
- Timing side channels are future work [S1].

## 9. Independent evidence
- Adoption: AWS documentation states Clean Rooms DP "uses the SampCert sampler, a proven correct sampler implementation developed by AWS" [S2].
- Peer review: PLDI 2025 with a "reviewed artifact" [S1].
- Bugs found in prior implementations: not quantified in the sections read; paper motivates with "bugs in basic mechanisms and random number generation have been a critical source of vulnerabilities in real-world DP systems" [S1].
- No independent third-party audit read.

## 10. Reproducibility
Not built: sandbox disk limit; queued for CI reproducibility job. Mathlib-heavy (requires `lake exe cache get`). Exact command:
```
curl https://raw.githubusercontent.com/leanprover/elan/master/elan-init.sh -sSf | sh -s -- -y --default-toolchain none
git clone --depth 1 https://github.com/leanprover/SampCert && cd SampCert
timeout 600 lake exe cache get && timeout 1500 lake build
```
The repo also ships `REPRODUCING.md`, a `Dockerfile` and `SampCertCheck.lean` [C1]. Static facts were collected from a `--depth 1` clone.

## 11. Generalisation
Pattern: "Lean is the implementation, with a dual-semantics primitive layer": a shallow-embedded monad whose primitives have a mathematical denotation for proofs and a tiny trusted native implementation (`@[extern]`) for execution; or an unverified syntactic translator to a mainstream language for deployment.

To apply to an arbitrary codebase: the algorithm must be rewritten in the Lean DSL (not verified in place); the trusted primitive layer must be small and auditable; deployment requires either linking Lean-compiled C or trusting a translator (here Lean→Dafny→Python). Case-specific preconditions: the code is small, self-contained, side-effect-free except randomness, has a black-box interface ("easier to deploy routines for generating random samples, since they have a simple interface") [S1], and uses exact rational arithmetic (no floats). It does not show verification of pre-existing code in another language, nor of interactive/stateful protocols.

## 12. Bearing on sub-claims (facts only, no verdict)
| Sub-claim | Fact from this reference | Source ID |
|---|---|---|
| H1 | Lean (with Mathlib) expresses probabilistic programs, infinite discrete distributions, unbounded loops and DP definitions (pure DP, zCDP) with a generic abstraction. No message passing or concurrency. | C1, S1 |
| H2 | 0 sorry/axiom/native_decide; ~12k lines of proof for privacy and sampler correctness using deep Mathlib results (Poisson summation). Guarantees are relative to add/remove-one neighbours on lists, user-supplied sensitivity hypotheses, and the mass-function model of the primitives (section 3a). | C1, S1 |
| H3 | Executable link via `@[extern]` C++ (57 trusted lines per paper) or an unverified Lean→Dafny→Python translator; deployed samplers in AWS Clean Rooms. | S1, S2, C1 |
| H4 | Requires re-implementing the algorithm in Lean's SLang; translator supports only "a limited subset of Lean". | S1 |
| H5 | Not directly applicable (no separate implementation is checked against the spec; the spec program is executed). | S1 |
| D | "Verified" = mechanised proof about mass-function semantics; "proven correct sampler" in AWS docs refers to this, with trusted FFI/translator in the loop. | S1, S2 |

## 13. Conflicts
- Trusted C++ size: paper "only 57 lines of C++" [S1] vs `ffi.cpp` 69 lines at commit `a2c53bb` [C1] (file includes headers and the extra `dirty_io_get` used for the Python API; counting method differs).
- Proof size: paper "over 12,000 lines of proof" [S1] vs 18,463 total Lean LOC at current commit [C1] (total includes definitions, extractor and later additions).
- Deployment path: README says deployed in Clean Rooms without naming the path [C1]; paper says deployment used the Dafny→Python translation [S1]. Consistent, recorded for completeness.

## 14. Sources
| ID | Title | URL | P/S | Depth | Summary |
|---|---|---|---|---|---|
| C1 | leanprover/SampCert repository @ a2c53bb (accessed 2026-11-07) | https://github.com/leanprover/SampCert | P | [CODE] | SLang monad with `@[extern]` primitives, samplers, DP framework, Dafny extractor; counts in section 6. |
| S1 | Verified Foundations for Differential Privacy (PLDI 2025, PACMPL 9 Art. 191), arXiv 2412.01671 (accessed 2026-11-07) | https://arxiv.org/abs/2412.01671 | P | [SECT] Abstract; §1 Introduction; §2 Abstract Differential Privacy (through Listing 3), §2.3-§2.5; "Obtaining Performant Samplers from Lean"; "Limitations and Future Work"; Conclusion; Data Availability Statement; Appendix "The Sparse Vector Technique"; Appendix "Extraction and Compilation Details". Every [S1] claim in this dossier is a verbatim quote from one of these sections | Design of SampCert, proofs, FFI and Dafny/Python extraction, deployment at AWS, limitations. |
| S2 | AWS Clean Rooms Differential Privacy (AWS documentation) (accessed 2026-11-07) | https://docs.aws.amazon.com/clean-rooms/latest/userguide/differential-privacy.html | P | [FULL] (whole page re-read without an extractor) | States Clean Rooms DP uses "the SampCert sampler, a proven correct sampler implementation developed by AWS". |

## 15. Unverified leads (excluded from evidence)
- Zenodo artifact DOI 10.5281/zenodo.11204806 (cited in README); not inspected.
- Possible follow-up work on Rényi DP / approximate DP extensions in later commits; not reviewed beyond counts.
