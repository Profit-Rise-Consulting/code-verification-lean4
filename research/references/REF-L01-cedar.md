# REF-L01: Cedar (cedar-spec Lean proofs, differential random testing vs Rust, SymCC/SymCert)

## 1. Identity
| Field | Value |
|---|---|
| Subject (protocol or algorithm) | Cedar authorization policy language: evaluator, authorizer, validator (typechecker), level-based entity slicing, typed partial evaluation (TPE), and the symbolic compiler SymCC used for SMT-based policy analyses (SymCert). Not a network protocol; an algorithmic/language-semantics reference. |
| Spec it targets (e.g. RFC 8446 section) | No external standard. The Lean model is itself the normative "definitional" specification of Cedar; Cedar RFCs (e.g. RFC-53, RFC-80, RFC-82, RFC 116) are the change process [S3]. |
| Organisation / authors | Amazon Web Services (Cedar team: Emina Torlak, Kesha Hietala, Craig Disselkoen, John Kastner, Andrew Wells and others) [S1][S2][S3][S4] |
| Primary URLs | https://github.com/cedar-policy/cedar-spec ; https://arxiv.org/abs/2407.01688 ; https://arxiv.org/abs/2403.04651 ; SymCert (FMCAD 2026) PDF https://cdn.amazon.science/9f/c9/5d18658a41c48628b686493aa327/scipub-approval152134-46496997-symcert-verifying-smtbased-policy-analyses.pdf |
| Repo, commit SHA, last commit date, licence | cedar-policy/cedar-spec @ `a4c5a89cd15637afc1dab39ef0c89baf31c7ebdf`, 2026-11-06 16:09:17 -0500, Apache-2.0 [C1] |
| Lean toolchain (or other tool and version) | `leanprover/lean4:v4.35.0` (root and `cedar-lean/lean-toolchain`); deps `batteries` and `doc-gen4` @ `v4.35.0`; no Mathlib [C1]. DRT harness: Rust + `cargo-fuzz`; solver: CVC5 [C1]. |
| Maturity (research prototype / production / abandoned / AI-generated) | Production. Release gating: "A new version of Cedar isn't released unless its model, proofs, and differential tests are up to date" [S4]. The Rust SymCC/analysis service "now runs in production" [S3]. Named AWS authors, papers, Apache-2.0 org project. |
| AI provenance | `none-found`. Basis: full commit history (blob:none clone, 796 commits) has 114 `Co-authored-by` trailers, all naming human contributors (e.g. Torlak, Hietala, Disselkoen); no AGENTS.md, CLAUDE.md, .claude, .cursor or Copilot config at a4c5a89 [C1]. |

## 2. Claim as stated by the authors (verbatim)
- "Developing a system with VGD involves writing an executable model of the system and mechanically proving properties about the model; writing production code for the system and using differential random testing (DRT) to check that the production code matches the model; and using property-based testing (PBT) to check properties of unmodeled parts of the production code." [S1]
- "While carrying out proofs, we found and fixed 4 bugs in Cedar's policy validator, and DRT and PBT helped us find and fix 21 additional bugs in various parts of Cedar." [S1]
- "If the two versions agree on the output, then we obtain a piece of evidence that Rust production code is on par with the Lean model." [S1]
- "SymCert provides four verified building blocks ... Together, these components let developers quickly build new analyses and prove them sound and complete with respect to Cedar's semantics." [S3]
- "Assuming the trusted solve function is correct, Theorem 7 and Theorem 8 ensure ..." soundness and completeness of the denies-all analysis [S3].
- "We adopted Lean ... because of its fast runtime, extensive libraries, and small trusted computing base (TCB)." [S4]

## 3. What is actually formalised
The "protocol" is a pure function from (request, entity store, policy set) to a decision. There is no state machine, message passing, concurrency or adversary model; the environment is the arbitrary input (request + entities), quantified universally in theorems.

Model definitions (README "Key definitions") [C1]: `evaluate` (Cedar/Spec), `satisfied`, `isAuthorized`; `typeOf`, `checkLevel` (Cedar/Validation); `SymCC.Compiler.compile`, `SymCC.Authorizer.isAuthorized` (Cedar/SymCC).

Theorem statements, verbatim, commit `a4c5a89`, paths relative to `cedar-lean/` [C1]:

`Cedar/Thm/Authorization.lean:69`
```lean
theorem forbid_trumps_permit
  (request : Request) (entities : Entities) (policies : Policies) :
  (IsExplicitlyForbidden request entities policies) →
  (isAuthorized request entities policies).decision = .deny
```
`Cedar/Thm/Authorization.lean:80`
```lean
theorem allowed_only_if_explicitly_permitted (request : Request) (entities : Entities) (policies : Policies) :
  (isAuthorized request entities policies).decision = .allow →
  IsExplicitlyPermitted request entities policies
```
`Cedar/Thm/Authorization.lean:158`
```lean
theorem order_and_dup_independent (request : Request) (entities : Entities) (policies₁ policies₂ : Policies) :
  policies₁ ≡ policies₂ →
  isAuthorized request entities policies₁ = isAuthorized request entities policies₂
```
`Cedar/Thm/Typechecking.lean:47`
```lean
theorem typecheck_is_sound (policy : Policy) (env : TypeEnv) (t : CedarType) (request : Request) (entities : Entities) :
  InstanceOfWellFormedEnvironment request entities env →
  typecheck policy env = .ok t →
  (∃ (b : Bool), EvaluatesTo policy.toExpr request entities b)
```
`Cedar/Thm/SymbolicCompilation.lean:71`
```lean
theorem compile_is_sound {x : Expr} {env : Env} {εnv : SymEnv} {t : Term.Outcome εnv} :
  εnv.WellFormedFor x →
  env.WellFormedFor x →
  env ∈ᵢ εnv →
  compile x εnv = .ok t →
  ∃ (o : Outcome Value),
    o ∈ₒ t ∧
    evaluate x env.request env.entities ∼ o
```
Further SymCC theorems in the same file: `compile_is_complete` (:99), `isAuthorized_is_sound` (:155), `isAuthorized_is_complete` (:183), `compile_well_typed` (:208).

`Cedar/Thm/Verification.lean:448`
```lean
theorem verifyEquivalent_is_sound  {ps₁ ps₂ : Policies} {εnv : SymEnv} {asserts : Asserts} :
  εnv.StronglyWellFormedForPolicies ps₁ →
  εnv.StronglyWellFormedForPolicies ps₂ →
  verifyEquivalent ps₁ ps₂ εnv = .ok asserts →
  εnv ⊭ asserts →
  ∀ env,
    env ∈ᵢ εnv →
    env.StronglyWellFormedForPolicies ps₁ →
    env.StronglyWellFormedForPolicies ps₂ →
    bothAllowOrBothDeny
      (Spec.isAuthorized env.request env.entities ps₁)
      (Spec.isAuthorized env.request env.entities ps₂)
```
Verification.lean has sound/complete pairs for `verifyNeverErrors` (:46, :64), `verifyAlwaysMatches`, `verifyNeverMatches`, `verifyMatchesEquivalent`, `verifyMatchesImplies`, `verifyMatchesDisjoint`, `verifyEquivalent` (:448, :471), `verifyDisjoint`, `verifyImplies`, `verifyAlwaysDenies` (:588, :611) and others. Note `εnv ⊭ asserts` ("unsatisfiable") is a Lean-level semantic predicate; the connection to an actual SMT solver answer is the trusted `solve` step (section 6).

Other proved properties listed in the README [C1]: sound policy slicing (`Cedar/Thm/Slicing.lean`), sound level-based entity slicing (`Cedar/Thm/Validation/Levels.lean`).

### 3a. Predicate semantics: what the quoted theorems do and do not guarantee
The meaning of each theorem depends on the predicates it uses. All definitions below are verbatim from commit `a4c5a89`, paths relative to `cedar-lean/` [C1].

`EvaluatesTo` (`Cedar/Thm/Validation/Typechecker/Basic.lean:61`):
```lean
def EvaluatesTo (e: Expr) (request : Request) (entities : Entities) (v : Value) : Prop :=
  evaluate e request entities = .error .entityDoesNotExist ∨
  evaluate e request entities = .error .extensionError ∨
  evaluate e request entities = .error .arithBoundsError ∨
  evaluate e request entities = .ok v
```
The docstring just above it (lines 39-60) says: "The `EvaluatesTo` predicate covers the (obvious) case where evaluation has no errors, but it also allows for errors of type `entityDoesNotExist`, `extensionError`, and `arithBoundsError`. The typechecker cannot protect against these errors because they depend on runtime information ... All other errors (`attrDoesNotExist` and `typeError`) can be prevented statically." The validator theorem uses the same predicate through `EvaluatesToBool` (`Cedar/Thm/Validation/Validator.lean:38`, `∃ (b : Bool), EvaluatesTo expr request entities b`). `validation_is_sound` (`Cedar/Thm/Validation.lean:55`) concludes `AllEvaluateToBool policies request entities`.

- Guarantees of `typecheck_is_sound` / `validation_is_sound`: if typechecking succeeds and the request and entities are instances of a well-formed environment, then evaluating the policy either yields a Boolean or fails with one of exactly three runtime errors (missing entity, extension error, arithmetic overflow). It can never fail with `attrDoesNotExist` or `typeError`, and it never returns a non-Boolean value.
- Not guaranteed: that the policy evaluates without error. A validated policy may still raise any of the three allowed errors at runtime. `typecheck_is_sound` as quoted also says nothing about which Boolean comes out, only that the result is Boolean-or-allowed-error.
- `InstanceOfWellFormedEnvironment` (`Cedar/Thm/Validation/Typechecker/Types.lean:146`) is `env.WellFormed ∧ InstanceOfRequestType request env ∧ InstanceOfSchema entities env`, so the theorem says nothing about requests or entity stores that fail schema validation.

`satisfied` and the authorizer (`Cedar/Spec/Authorizer.lean:29-30, 56-62`):
```lean
public def satisfied (policy : Policy) (req : Request) (entities : Entities) : Bool :=
  (evaluate policy.toExpr req entities) = .ok true
...
public def isAuthorized (req : Request) (entities : Entities) (policies : Policies) : Response :=
  let forbids := satisfiedPolicies .forbid policies req entities
  let permits := satisfiedPolicies .permit policies req entities
  let erroringPolicies := errorPolicies policies req entities
  if forbids.isEmpty && !permits.isEmpty
  then { decision := .allow, determiningPolicies := permits, erroringPolicies }
  else { decision := .deny,  determiningPolicies := forbids, erroringPolicies }
```
`HasSatisfiedEffect` (`Cedar/Thm/Authorization.lean:37`) is `∃ policy, policy ∈ policies ∧ policy.effect = effect ∧ satisfied policy request entities`; `IsExplicitlyForbidden := HasSatisfiedEffect .forbid` (:50) and `IsExplicitlyPermitted := HasSatisfiedEffect .permit` (:60).

- Guarantee of `forbid_trumps_permit`: if some forbid policy evaluates to `.ok true`, the decision is deny.
- Not guaranteed: a forbid policy that evaluates to an error is not "satisfied". It is recorded in `erroringPolicies` but does not count as a forbid. If a forbid policy errors at runtime (for example with one of the three errors `EvaluatesTo` allows) while some permit policy evaluates to `true`, the decision is allow. The theorem does not say "any applicable forbid policy blocks the request"; it covers only forbid policies that evaluate to `true` without error.
- Guarantee of `allowed_only_if_explicitly_permitted`: an allow decision implies that at least one permit policy evaluated to `.ok true`. It does not say that no policy errored, and it does not say that the forbids were checked successfully (erroring forbids are skipped as above).
- `≡` in `order_and_dup_independent` is `a ⊆ b ∧ b ⊆ a` (`Cedar/Thm/Data/List/Basic.lean:40`). The conclusion is equality of the whole `Response`, including `determiningPolicies` and `erroringPolicies`.

SymCC/SymCert relations (`Cedar/Thm/SymCC/Data/Basic.lean`):
```lean
public def SameOutcomes (res : Result Value) (out : Outcome Value) : Prop :=
  match res, out with
  | .ok v₁,   .ok v₂    => v₁ = v₂
  | .error e, .error () => e ≠ .entityDoesNotExist
  | _, _                => False
```
(line 222; installed as `∼` for `Result Value`/`Outcome Value` at line 241). `∈ₒ` (`memOfTermOutcome`, :782-785) and `∈ᵢ` (`memOfSymEnv`, :787-790) both mean "there exists a well-formed interpretation `I` such that ... `∼` ... `interpret I`". `⊭` is `Asserts.Unsatisfiable` (:766-770), i.e. `¬ ∃ I, I.WellFormed env.entities ∧ asserts.satisfiedBy I`.

- `compile_is_sound`: the symbolic outcome matches concrete evaluation exactly on values, but all errors collapse to a single symbolic `.error ()`. The error kind is not preserved, and a concrete `entityDoesNotExist` error is excluded by the relation, so the theorem relies on the well-formedness premises to rule that error out.
- `verifyEquivalent_is_sound`: its premise `εnv ⊭ asserts` is the mathematical statement that no well-formed interpretation satisfies the assertions. It is not the statement "CVC5 answered unsat". The bridge to a solver answer is the trusted `solve` function [S3]. The conclusion `bothAllowOrBothDeny` (`Cedar/Thm/SymCC/Verifier/VerifyIsAuthorizedQueries.lean:27`) is `r₁.decision = r₂.decision`: equal decisions only, with no claim about equal `determiningPolicies` or `erroringPolicies`.
- The conclusion holds only for environments satisfying `env.StronglyWellFormedForPolicies` (`Cedar/Thm/SymCC/Data/Hierarchy.lean:123`). This requires `Entities.StronglyWellFormed := es.WellFormed ∧ es.Hierarchical` (:105), where `Hierarchical` is `Acyclic ∧ Transitive` (:92-103). Entity stores with cyclic or non-transitively-closed ancestor sets are outside the guarantee; SymCert states this explicitly ("without it, nearly all interesting properties can be violated by ill-formed environments") [S3].

SymCert decomposition [S3]: compiler correctness split into "reducibility (the compiler partially evaluates literal inputs) and a property we call interpretability (compilation commutes with SMT evaluation)", plus a hierarchy enforcer (acyclicity/transitivity constraints) and a counterexample extractor that converts infinite SMT models into finite Cedar inputs.

## 4. Property classes covered
| Class | Status | Evidence |
|---|---|---|
| Functional correctness | partial | The model is the definition; theorems are properties of it (forbid-trumps-permit, order independence) plus correctness of derived algorithms against the model: typechecker soundness, slicing soundness, SymCC soundness and completeness w.r.t. `evaluate` [C1]. The Rust implementation's functional correctness is not proved; it is tested by DRT [S1]. |
| Safety invariant | proved (security properties of the decision function) | `forbid_trumps_permit`, `allowed_only_if_explicitly_permitted` [C1]. Scope: these cover only policies that evaluate to `.ok true`; erroring policies are skipped by the authorizer (section 3a). |
| Liveness | n/a | Not applicable; only termination of all model functions, enforced by Lean's totality checking; termination proofs found a validator non-termination bug [S1]. |
| Refinement | partial | Proofs that one Lean algorithm agrees with another Lean definition (slicing = full authorization, SymCC semantics = concrete semantics) [C1]. Lean-to-Rust refinement is tested, not proved [S1]. |
| Symbolic security | no | No adversary model. |
| Computational security | no | |
| Parsing | tested | "our parsers are not modeled in Lean" [S1]; parser tested only by PBT [S1]. |
| Constant-time | no | |
| Other | proved | Sound and complete reduction of policy analyses to SMT (SymCert) [C1][S3]. |

## 5. Link to implementation code
Mechanism: differential random testing (DRT) plus property-based testing (PBT). The Lean model is compiled to native code and exposed via `cedar-lean/DiffTest/Main.lean` ("This file defines the public interfaces for the Lean implementation. The input and output are stringified JSON objects"; requests arrive as binary protobuf) [C1]. Rust `cargo-fuzz` targets in `cedar-drt/fuzz/fuzz_targets` (84 entries at this commit) generate random policies, entities, requests and schemas via `cedar-policy-generators` and compare the Rust `cedar` crates with the Lean model; the cedar-drt README table labels each target DRT or PBT (e.g. `abac`, `abac-type-directed`, `rbac`, `validation-drt`, `entity-validation`, `tpe-is-authorized-drt`, `symcc-term-drt-always-allows`, `symcc-cex-drt`) [C1]. The Rust SymCC crate `cedar-policy-symcc` is a fuzz dependency (`cedar-drt/fuzz/Cargo.toml:25,55`) [C1].

Fuzzing budget reported: "We allocate 4096 CPU units (4 vCPUs) and 8GB memory to fuzz each target for 6 hours." [S1] Daily runs: "The Rust service now runs in production, while the original Lean implementation serves as a correctness oracle for daily differential random testing." [S3]

Guarantee: statistical/empirical, not proof. Authors: agreement gives "a piece of evidence" [S1]. Testing missed the validator non-termination bug ("the probability of generating inputs triggering this bug is extremely low") and is "inherently unable to find bugs outside the scope of the test generators" [S1].

Alternatives rejected by the authors: deploying Lean-generated C ("there are significant downsides to writing production applications in Lean and deploying its generated C code ... C is a memory-unsafe language, so a bug in the Lean compiler could lead to security issues") and Rust verification tools ("The model consumable by one of these tools is unlikely to be as readable as the Lean model, due to Rust's low-level nature") [S1].

## 6. Trusted computing base and soundness hygiene
Counts over all `.lean` files in `cedar-lean/` excluding `.lake`, non-comment lines, word-boundary match [C1]:

| Token | Count | Location / note |
|---|---|---|
| sorry | 0 | |
| admit | 0 real | Textual hits are the English word "admits"/"admit" in docstrings, `Cedar/Thm/Data/String.lean:36,55`. |
| axiom | 0 | |
| native_decide | 26 | 10 in proof/model tree: `Cedar/Thm/SymCC/Data/Ext.lean` (8, Int64 constants such as `msPerDay_eq ... := by native_decide`, line 36) and `Cedar/Data/Int64.lean` (2, lines 113, 118); 16 in `UnitTest/` (IPAddr 8, Decimal 8). The 10 in the proof tree add the Lean compiler/`Lean.ofReduceBool` to the TCB of those theorems. |
| implemented_by | 2 | `CedarFFI/Main.lean:296,305` (`solve`, `timedSolve` wrappers around the solver IO), not in the proof tree. |
| extern | 0 | |
| opaque | 4 hits; 2 declarations | `CedarFFI/Main.lean:297,306`; other hits are comments. |
| partial | 139 textual; 41 `partial def` | 39 in `CedarProto/` (protobuf deserialisation used by DiffTest), 1 in `Cedar/SymCC`, 1 in `UnitTest`. |

Trusted components beyond the Lean kernel:
- The SMT solver: "An SMT solver [3] checks satisfiability via the trusted solve function"; Table I lists "Solver (trusted) 566 [model LOC] 47 [hours] 0 0" [S3]. The solver is CVC5, launched as an external process from the `CVC5` environment variable (`Cedar/SymCC/Solver.lean:27-29,120`) [C1].
- The Rust production code is outside the proof; only DRT links it [S1].
- The DiffTest JSON/protobuf interface (`CedarProto`, `partial def`) and Rust-side conversions are unverified glue [C1].
- Parsers (Rust) are unmodeled [S1].
- Lean compiler for executing the model during DRT; `native_decide` uses in Int64 lemmas.

## 7. Scale, effort, automation
- Lean LOC at `a4c5a89`: 126,845 total in 388 files; 2,595 `theorem`/`lemma` declarations [C1]. By directory: Cedar/Spec 2,239; Cedar/Validation 2,276; Cedar/SymCC 4,545; Cedar/TPE 1,215; Cedar/Data 1,023; Cedar/Thm 97,292 (226 files, 2,569 theorems); CedarProto 5,356; UnitTest 4,019; SymTest 3,977; CedarFFI 1,262; DiffTest 173 [C1]. Rough spec:proof ratio ≈ 11k model : 97k proof.
- Historic (2024) table: Lean model 1,673 LOC, Lean proofs 5,714, Rust prod 15,693, Rust tests 20,458 [S1]; "this proof [validator soundness] took 18 person-days" (4,686 LOC) [S4]; "Over the past 6 months, we built a formal model of Cedar in Lean" [S4].
- SymCert Table I totals (cloc, person-hours): model 2,239 LOC / 158 h; proof 21,747 LOC / 505 h; compiler 318 model LOC with "symbolic compiler (81 hours) and its proof (327 hours)"; "The overall verification effort for SymCert is 5:1 in person-hours spent on proofs versus models"; five analyses implemented "in less than an hour using 39 lines of code", proofs "7 hours and 512 lines" [S3]. Feature extensions: tags patch 1,261 lines in 4 days; "A developer new to Lean extended SymCert to support the RFC in just 8 days" [S3].
- Performance: Lean authorizer median 6 µs vs Rust 10 µs during DRT [S1]; "185 seconds in total to check all proofs and compile models" (2024) [S4].
- Automation: interactive tactics (`simp`, `omega`, custom simp sets); no SMT inside proofs; random generators with type-directed generation [S1].
- Build time: not built here (section 10).

## 8. Limitations stated by the authors (verbatim)
- "Unsurprisingly, Cedar's testing framework has missed bugs, too. The most notable example is that it did not discover the non-termination bug described in Section 3.1." [S1]
- "Our framework is also inherently unable to find bugs outside the scope of the test generators. For example, we did not discover some parser bugs triggered by malformed policies because our test generators create abstract syntax trees ..." [S1]
- "our parsers are not modeled in Lean because there is currently no library support for parser generators, and it is unclear what properties we could prove about a parser model." [S1]
- "For Cedar DRT, we found that even when the ABAC policy generator achieves full line coverage, many of the generated inputs exercise error handling code. Furthermore, 35.5% of the when condition-expressions generated ... are boolean literals" [S1]
- "Lean requires all recursive definitions to be well-founded ... which complicates modeling of complex recursive structures." [S1]
- "There are still some small challenges with Lean, such as its sparse documentation and sluggish termination checking for mutual induction." [S4]
- SymCert soundness holds "Assuming the trusted solve function is correct" [S3]; all analyses are relative to a closure/well-formedness assumption: "without it, nearly all interesting properties can be violated by ill-formed environments" [S3].

## 9. Independent evidence
- Bugs: 4 validator bugs found by proofs and 21 by DRT/PBT (2024), e.g. a bug in a Rust IP-address parsing package found by DRT [S1]. OOPSLA paper: DRT "uncovering nearly two dozen bugs since the project's inception" [S2]. SymCert: subtle hierarchy/error-propagation bugs "Our proofs caught" [S3].
- Adoption: the AWS blog post is filed under Amazon Verified Permissions, the managed service using Cedar [S4]; Rust SymCC analysis service in production [S3].
- Reviews: all sources are by the authors' team; peer-reviewed venues (FSE 2024 Companion industry track, OOPSLA 2024, FMCAD 2026). No independent third-party audit read.

## 10. Reproducibility
Not built: sandbox disk limit; queued for CI reproducibility job. Exact command (from `cedar-lean/README.md`):
```
curl https://raw.githubusercontent.com/leanprover/elan/master/elan-init.sh -sSf | sh -s -- -y --default-toolchain none
git clone --depth 1 https://github.com/cedar-policy/cedar-spec && cd cedar-spec/cedar-lean
lake update && timeout 1500 lake build Cedar
```
DRT additionally needs the `cedar` Rust repo cloned inside `cedar-spec`, `source cedar-drt/set_env_vars.sh`, `../cedar-drt/build_lean_lib.sh`, `cargo build`, then `cargo fuzz run -s none <target>`; "The build has only been tested on Amazon Linux 2" (root README) [C1]. Static facts above were collected from a `--depth 1` clone.

## 11. Generalisation
Pattern: "verification-guided development" (VGD): executable Lean model as reference semantics + proofs on the model + differential random testing of an independent production implementation in a mainstream language (Rust) + PBT for unmodeled parts. Second pattern (SymCert): a verified reduction from a decidable language to SMT, with the solver as the only trusted analysis component.

To apply to an arbitrary codebase one needs: (a) a deterministic, executable Lean model of the component's input-output behaviour; (b) a serialisation boundary (JSON/protobuf) both sides can consume; (c) input generators that reach deep behaviour (type-directed generators were needed); (d) CI gating that keeps model, proofs and tests synchronised.

Case-specific preconditions: Cedar is a small, deterministic, terminating, side-effect-free language designed "to support sound and complete reduction of policy semantics to a decidable fragment of SMT-LIB" [S3]; there is no concurrency, network, timing or adversary to model; the same team owns both model and implementation. For a stateful network protocol, DRT would require driving both sides through message sequences (trace-level comparison), which this reference does not demonstrate.

## 12. Bearing on sub-claims (facts only, no verdict)
| Sub-claim | Fact from this reference | Source ID |
|---|---|---|
| H1 | Lean expresses a full industrial authorization-language semantics, typechecker and SMT encoding; no state machine, concurrency, message passing or adversary is modelled. | C1, S1 |
| H2 | ~97k lines of Lean proofs, 2,569 theorems, 0 sorry/axiom; SymCC sound and complete; proof effort reported in person-hours (505 h SymCert proofs). Properties are about a pure function, not a protocol. Their strength depends on the predicates (section 3a): type soundness via `EvaluatesTo` allows three runtime errors; forbid-trumps-permit applies only to forbid policies that evaluate to `true`; SymCC treats all errors as one; analysis results assume acyclic, transitive hierarchies and compare decisions only. | C1, S3 |
| H3 | Lean model linked to Rust production code by DRT/PBT only; authors describe agreement as "a piece of evidence"; DRT missed a non-termination bug. Release gated on model/proofs/DRT up to date. | S1, S4 |
| H4 | Mechanism depends on an executable model plus a serialisation interface plus random generators; build "only been tested on Amazon Linux 2". Language-agnostic in principle (any implementation callable from the fuzz harness). | C1, S1 |
| H5 | The implementation is checked against the Lean spec by testing, independently of the protocol-level proofs: DRT works on the executable model regardless of which theorems exist; PBT tests properties "we have yet to prove". | S1 |
| D | "Verify" = mechanised proof about the Lean model; "check"/"match" for Rust = differential random testing; SymCert "sound and complete" analyses rely on a trusted solver. | S1, S3 |

## 13. Conflicts
- Rust LOC: the AWS blog reports Rust production code 24,915 LOC (Evaluator/Authorizer 13,664, Validator 11,251) [S4]; the FSE paper reports Rust prod 15,693 with separate Rust tests 20,458 and parser 4,114 [S1]. Different counting (tests and diagnostics included or not); not resolved here.
- Execution speed: blog "On average, Lean takes 5 microseconds ... compared to 7 microseconds for Rust" [S4]; paper "median execution time for the Lean authorizer is 6 microseconds, compared to 10 microseconds for Rust" [S1] (average vs median, different dates).
- Bug counts: 4 proof-found + 21 DRT/PBT [S1] vs "nearly two dozen bugs" [S2]; consistent in order of magnitude.
- Statement of `forbid_trumps_permit`: 2024 sources show an explicit `∃ policy ∈ policies ∧ effect = forbid ∧ satisfied` premise [S1][S4]; current repo uses `IsExplicitlyForbidden` (Authorization.lean:69) [C1]. Restatement, not necessarily a semantic change (not checked).
- Validation soundness, paper vs Lean: the OOPSLA paper states informally (§3.5) "Validation soundness: ... If ∅, Γ𝐴 ⊢ 𝑒𝐴 : Bool, 𝜀 under entity type map 𝑀, then 𝜇, 𝜎 ⊢ 𝑒𝐴 −→∗ 𝑣 for all entity stores 𝜇 conforming to 𝑀, and for which entity references in 𝑐 and 𝜎 are defined" (evaluation reaches a value) [S2]; the Lean conclusion `EvaluatesTo` (`Cedar/Thm/Validation/Typechecker/Basic.lean:61`, see section 3a) also admits the `entityDoesNotExist`, `extensionError` and `arithBoundsError` error results [C1]. Not resolved here.
- SymCC proof status: OOPSLA paper (2024) says "proofs of soundness and completeness using the Lean encoder are underway" [S2]; current repo contains them [C1][S3]. Time difference.

## 14. Sources
| ID | Title | URL | P/S | Depth | Summary |
|---|---|---|---|---|---|
| C1 | cedar-policy/cedar-spec repository @ a4c5a89 (accessed 2026-11-07) | https://github.com/cedar-policy/cedar-spec | P | [CODE] | Lean model, proofs (Cedar/Thm), DiffTest interface, CedarProto, cedar-drt fuzz harness with 84 targets; counts in section 6. |
| S1 | How We Built Cedar: A Verification-Guided Approach (FSE 2024 Companion), arXiv 2407.01688 (accessed 2026-11-07) | https://arxiv.org/abs/2407.01688 | P | [FULL] | Describes VGD: Lean model + proofs + DRT/PBT vs Rust; LOC tables, 4+21 bugs, missed bugs and limitations. |
| S2 | Cedar: A New Language for Expressive, Fast, Safe, and Analyzable Authorization (OOPSLA 2024, extended), arXiv 2403.04651 (accessed 2026-11-07) | https://arxiv.org/abs/2403.04651 | P | [SECT] front matter, §3.5 Properties, §3.6 Full Cedar, §4 Symbolic Compilation (4.1-4.4) | Language design paper. Quoted: "This testing regime has been fruitful, uncovering nearly two dozen bugs since the project's inception, and forcing us (via CI) to keep the Lean spec and proofs up to date." (§3.6); "We have proved soundness and completeness of the encoding on paper. We have also implemented the symbolic encoder in Lean, and proofs of soundness and completeness using the Lean encoder are underway, as of this writing." (§4.4). |
| S3 | SymCert: Verifying SMT-Based Policy Analyses (FMCAD 2026), E. Torlak (accessed 2026-11-07) | https://cdn.amazon.science/9f/c9/5d18658a41c48628b686493aa327/scipub-approval152134-46496997-symcert-verifying-smtbased-policy-analyses.pdf | P | [FULL] | Verified symbolic compiler, enforcer and extractor for sound/complete SMT analyses; trusted solver; effort tables; Rust port in production with Lean as DRT oracle. |
| S4 | Lean Into Verified Software Development, AWS Open Source Blog, 2024-04-08 (accessed 2026-11-07) | https://aws.amazon.com/blogs/opensource/lean-into-verified-software-development/ | P | [FULL] | Practitioner account: LOC table, 18 person-days validator proof, 185 s proof check, release gating on proofs and DRT. |

## 15. Unverified leads (excluded from evidence)
- Amazon Science blog "How we built Cedar with automated reasoning and differential testing" (linked from cedar-spec README); not read.
- AWS "Introducing Cedar Analysis" announcement / open-sourced Cedar Analysis CLI (2025); not read.
- Earlier Dafny version of the Cedar model (the team moved from Dafny to Lean; mentioned in secondary commentary); not verified here.
- Amazon Science publication page for SymCert (search snippet) used only to locate the PDF.
