# REF-L07: Aeneas (Rust to Lean via Charon) and Microsoft SymCrypt Lean proofs

Access date for all sources: 2026-11-07. Source IDs refer to section 14. Admissible sources: [FULL] 6 (A1, A3, A4, A6, A7, A8), [CODE] 4 (C1, C2, C3, C4). Phase 1c (2026-11-07): abstract-only sources A2 and A5 moved to section 15; A6 re-read as the full PDF text without an extractor; `spec` semantics anchored to the pinned Aeneas revision d71d2e3f (C4); section 3a added.

## 1. Identity
| Field | Value |
|---|---|
| Subject (protocol or algorithm) | (a) Aeneas: a general Rust-to-Lean (and F*, Coq/Rocq, HOL4) functional translator, consuming Charon's LLBC. (b) SymCrypt verified Rust: cryptographic primitives SHA-3/SHAKE (FIPS 202) and ML-KEM (FIPS 203) in the public release; the paper also reports AES/AES-GCM, SHA-2, HMAC/HKDF, ML-DSA, FrodoKEM, HPKE (most marked "experimental"). These are algorithms/primitives, not interactive protocols. HPKE (RFC 9180) is the most protocol-like component and is not in the public release. |
| Spec it targets | FIPS 202 (SHA-3), FIPS 203 (ML-KEM); paper adds FIPS 197 / SP 800-38D, FIPS 180-4, RFC 2104/5869, FIPS 204, draft-frodokem-02, RFC 9180 [A1, Fig. 4]. |
| Organisation / authors | Aeneas/Charon: Son Ho, Jonathan Protzenko, Aymeric Fromherz, Guillaume Boisseau and others (Inria, Microsoft) [A3, A6]. SymCrypt paper: Son Ho, Cédric Fournet, Jonathan Protzenko, Michael Naehrig, Joshua Clune, Patrick Longa, Guillaume Boisseau, Fernando Leal Sánchez, Aymeric Fromherz, Antoine Delignat-Lavaud (Microsoft, Google, CMU, Inria, ENS Paris-Saclay) [A1]. |
| Primary URLs | https://arxiv.org/abs/2609.15648 ; https://github.com/microsoft/SymCrypt/tree/feature/verifiedcrypto ; https://github.com/AeneasVerif/aeneas ; https://github.com/AeneasVerif/charon |
| Repo, commit SHA, last commit date, licence | SymCrypt `feature/verifiedcrypto` @ `c2e575ace0ea4b6b7a4184c1f19b81d1d5b2b5be`, 2026-07-12, MIT (single squashed commit "Merged PR 16147371: Publish ML-KEM and SHA3 proofs", author Samuel Lee). Aeneas `main` @ `c5eb4a28cef65e0aa27743105308243e535a2a2b`, 2026-11-07, Apache-2.0. Charon `main` @ `f070bb9233091640c777447cdaef594845d6c3b8`, 2026-11-06. Aeneas pinned revision used by SymCrypt @ `d71d2e3f2cb763f7faf15ab606ac9cf32da8dead`, 2026-07-11 ("Update Charon (#1202)"), `lean-toolchain` = `leanprover/lean4:v4.31.0` [C4]. |
| Lean toolchain | SymCrypt: `leanprover/lean4:v4.31.0`, pinned to Aeneas `d71d2e3f`, Charon `5a501733`, rustc `nightly-2026-06-01` (`SymCRust/lean/Symcrust/Code/EXTRACTION.md`). Aeneas main backend: `leanprover/lean4:v4.35.0`; `charon-pin` = `8cae8fd1905946de61147a65ecdd7122cc6b491b`. |
| Maturity | Aeneas: research tool used in production verification. SymCrypt SHA-3 and ML-KEM: production code; the MSR blog says it is "being used in insiders builds of Windows today" [A4]. Other algorithms in the paper: "experimental", "we do not claim our experimental code is production-ready" [A1 §1]. Proofs are largely AI-agent-generated but kernel-checked, with human review of specs and top-level statements (stated by the authors, see section 7). |
| AI provenance | `author-reported`. Basis: A1 §8.1 states that "agents generated ∼5.1k theorems for a total of 125 KLOC of proofs" and names the models used (section 7). Not independently established. |

## 2. Claim as stated by the authors (verbatim)
- "Our 237 KLOC Lean development establishes safety, panic-freedom, and functional correctness of 16.7 KLOC of Rust code supporting post-quantum cipher suites for x86-64 and ARM platforms." [A1, abstract]
- "We verify its implementations of algorithms such as SHA-3 and ML-KEM, which were ported from C to Rust." [A1, abstract]
- "Verified means that every public function is equipped with a theorem that specifies it in terms of the algorithms in the spec, that their proofs are complete and valid, and that the formal specification itself has been tested and reviewed against the standard." [C1 README-VERIFIEDCRYPTO.md §2]
- "This first release includes complete proofs for the Rust ML-KEM and SHA3 code that is being used in insiders builds of Windows today." [A4]
- Aeneas soundness (ICFP 2024), as summarised by a secondary source read in full: "Ho, Fromherz and Protzenko prove that LLBC, a borrow-centric model of Rust, can be equipped with a symbolic semantics that plays the role of a borrow-checker. They also prove that LLBC is a sound model with respect to an operational semantics over a standard memory model à la CompCert." [A6, abstract]. The ICFP 2022/2024 papers themselves were read only at abstract level (section 15).

## 3. What is actually formalised
### 3.1 Aeneas pipeline and semantics of the Lean model
- Charon drives rustc and exports MIR as ULLBC/LLBC; Aeneas symbolically executes LLBC, computing a borrow graph, and emits a pure functional program; mutable borrows become values returned to the caller, and borrows returned from functions become "backward functions" [A1 §5.1; A3 §4.3].
- Paper's intended semantics: "The monadic postcondition notation also asserts that the extracted computation terminates without panic." [A1 App. C]. The paper describes `Result` as an exception monad in which `←` propagates failure [A1 §5.1].
- Code semantics at the pinned revision d71d2e3f (the revision SymCrypt's EXTRACTION.md names) [C4]. `Result` is a plain inductive with three constructors, `backends/lean/Aeneas/Std/Primitives.lean:63-67`:
  ```lean
  inductive Result (α : Type u) where
    | ok (v: α): Result α
    | fail (e: Error): Result α
    | div
  deriving Repr, BEq
  ```
  where `Error` (Primitives.lean:51-58) is `assertionFailure | integerOverflow | divisionByZero | arrayOutOfBounds | maximumSizeExceeded | panic | undef`. The triple is defined in `backends/lean/Aeneas/Std/WP.lean:11-29`:
  ```lean
  def Post α := (α -> Prop)
  def Pre := Prop
  def Wp α := Post α → Pre
  def wp_return (x:α) : Wp α := fun p => p x
  def theta (m:Result α) : Wp α :=
    match m with
    | ok x => wp_return x
    | fail _ => fun _ => False
    | div => fun _ => False
  def spec {α} (x:Result α) (p:Post α) :=
    theta x p
  def dspec {α} (x:Result α) (p:Post α) :=
    match x with
    | ok x => p x
    | fail _ => False
    | div => True
  ```
  The notation `e ⦃ x => p ⦄` expands to `Aeneas.Std.WP.spec e post` and `e ⦃ x => p ⦄div` to `dspec` (three `macro_rules` blocks at WP.lean:354, 363 and 372). `theorem spec_ok (x : α) : spec (ok x) p ↔ p x` is in WP.lean within lines 50-60. So, at d71d2e3f, a proved `f args ⦃ post ⦄` states that the Lean term `f args` reduces to `ok v` with `post v`: it is neither `fail e` (any of the seven `Error` cases) nor `div`. This is the code semantics; it matches the paper's stated intent, but it is a statement about the Aeneas-generated Lean model, not about rustc-compiled machine code (section 5). All SymCrypt theorems quoted below use the `⦃ ⦄` (not `⦃ ⦄div`) form. Note that `div` is the Lean model's representation of divergence (for `partial_fixpoint` definitions); a theorem excluding `div` is a termination statement about the model.
- Aeneas `main` (c5eb4a2, Lean v4.35.0) has since replaced this with an interaction-tree `Result` (`abbrev Result := ITree RustEffect`) and a `DWP`-based `spec` at `WP.lean:56`, with `spec_fail ... ↔ False` (WP.lean:103) and `spec_div ... ↔ False` (WP.lean:106) [C2]. That main-branch definition is not the one SymCrypt's proofs were checked against and is recorded here only to show that the two revisions differ.
- Rust scalars are subtypes, e.g. U32 as `{x : Nat // x ≤ U32.max}`; arrays as subtypes of `List`; `assert!` becomes `massert`, so dynamic assertions must be discharged statically [A1 §5.5].

### 3.2 SymCrypt: how the algorithm is modelled
Specifications live in `SymCRust/lean/Spec/` and are written independently of the code, as executable Lean mirroring FIPS pseudocode line by line (paper Fig. 2 shows the NTT) [A1 §4]. Specs are tested against NIST CAVP vectors: `Spec/MLKEM/REPORT.md` lists 14 micro-tests plus 3 key-generation and 3 encapsulation vector sets [C1]. Randomness is modelled as an underspecified tape (`MLKEM.RandomTape`), introduced by an axiom on the FFI `random` call (section 6).

Top-level ML-KEM theorems (verbatim, commit c2e575a):

`SymCRust/lean/Symcrust/Properties/MLKEM/Decaps.lean:1701`
```lean
@[step]
theorem mlkem.decapsulate.spec
    (params : ParameterSet)        -- Spec-level parameter set
    (pk_mlkem_key : mlkem.key.Key) -- Input: private key container
    (pb_ciphertext : Slice U8)     -- Input: encapsulated key
    (pb_agreed_secret : Slice U8)  -- Output: shared secret
    (h_key : wfDecapKey pk_mlkem_key params) :
    mlkem.decapsulate pk_mlkem_key pb_ciphertext pb_agreed_secret
      ⦃ error pb_agreed_secret' =>
          match error with
          | .NoError =>
              ∃ h_s h_c,
                MLKEM.Decaps params (pk_mlkem_key.toDecapKey params)
                    (pb_ciphertext.toSpec _ h_c)
                  = some (pb_agreed_secret'.toSpec _ h_s)
          | .InvalidArgument =>
              pb_agreed_secret.length ≠ 32 ∨
              pb_ciphertext.length ≠ cipherlength params
          | .MemoryAllocationFailure =>
              out_of_memory
          | _ => False
          ⦄
```
(comment lines omitted; full text lines 1701-1733.)

`Symcrust/Properties/MLKEM/Encaps.lean:656` (opening):
```lean
theorem mlkem.encapsulate.spec
    (params : ParameterSet)
    (pk_mlkem_key : mlkem.key.Key)
    (pb_agreed_secret pb_ciphertext : Slice U8)
    (h_key : wfEncapKey pk_mlkem_key params) :
    mlkem.encapsulate pk_mlkem_key pb_agreed_secret pb_ciphertext
      ⦃ error pb_agreed_secret' pb_ciphertext' =>
          match error with
          | .NoError =>
              ∃ (tape : MLKEM.RandomTape)
                (h_a : pb_agreed_secret'.length = 32)
                (h_c : pb_ciphertext'.length = cipherlength params),
                match MLKEM.Encaps params
                        (pk_mlkem_key.toPubKey params) tape with
                | some (K, c, _) => ...
```

`Symcrust/Properties/MLKEM/KeyGen.lean:261` (opening):
```lean
theorem mlkem.key_generate.spec
    (params : ParameterSet)
    (pk_mlkem_key : mlkem.key.Key) (flags : U32)
    (h_wf : wfKey pk_mlkem_key params) :
    mlkem.key_generate pk_mlkem_key flags
      ⦃ err key' =>
          wfKey key' params ∧
          match err with
          | Error.NoError =>
              wfPrivateSeed key' params ∧
              ∃ (tape : MLKEM.RandomTape),
                let (ek, dk, _) := MLKEM.KeyGen params tape
                ek = key'.toEncapKey params ∧
                dk = key'.toDecapKey params
          | Error.InvalidArgument => ...
```

SHA-3/SHAKE, stateful API with ghost state (`Symcrust/Properties/SHA3/StatefulXof.lean:71`):
```lean
@[step] theorem ShakeState.extract.spec
    {R : Std.Usize} {B : Std.U32}
    (self : ShakeState R B) (result : Slice Std.U8) (wipe : Bool)
    (g : GhostState)
    (h : absorbing self.state g ∨ squeezing self.state g) :
    ShakeState.Insts.SymcrustHashStatefulXof.extract self result wipe
    ⦃ (r : ShakeState R B × Slice Std.U8) =>
      r.2.length = result.length ∧
      r.2.val = (extractOutput g result.length).toList ∧
      (if wipe then absorbing r.1.state (.init g.rate g.padVal g.h_rate)
       else squeezing r.1.state (g.squeeze r.2.val)) ⦄ := by
  unfold ShakeState.Insts.SymcrustHashStatefulXof.extract
  step*
  exact ⟨ks_post1, ks_post2, ks_post3⟩
```
One-shot SHAKE: `OneShotShake.xof.spec` at `StatefulXof.lean:118`.

Specification-level properties: `theorem KECCAK_f_bijective : Function.Bijective KECCAK_f` (`Spec/SHA3/Permutation.lean:492`); `theorem kyber_terminates (B : 𝔹 34) : ∃ (n : Nat), (sampleNttPartial B n).2 = 256` (`Symcrust/Properties/MLKEM/Sampling/SampleNTTBridge.lean:111`), proved via `Helpers.sampleNttPartialAux_terminates` and `Spec/SHA3/Termination.lean` (a formalisation of Barbosa and Schwabe, ePrint 2023/708). The paper says the NTT ring-isomorphism and decryption-failure bounds are also proved [A1 §4]; a grep of `Spec/` at this commit for `RingEquiv`, `≃+*` or failure-probability terms found no such theorem (see section 13).

Hardware intrinsics: Rust shims in `src/verify/intrinsics/` (lane-array models of SSE2, SSSE3, AVX2, AES-NI, SHA-NI, PCLMULQDQ, NEON, Armv8 AES), verified like other code, plus Lean axioms in `Intrinsics/Axioms/` for AES/SHA round instructions. Paper Fig. 3: 122 shims with Rust bodies, 11 opaque [A1 §6]. Multi-target: Charon is run once per target and LLBC merged, turning `cfg` into a dynamic branch on an underspecified `get_target` [A1 §5.7, App. A].

## 3a. Predicate semantics
All files are under `SymCRust/lean/` at SymCrypt c2e575a [C1]; the triple `⦃ ⦄` means `Aeneas.Std.WP.spec` at d71d2e3f (section 3.1) [C4].

Conclusion predicate of `mlkem.decapsulate.spec` (Decaps.lean:1701-1733): `MLKEM.Decaps`, defined at `Spec/MLKEM/Spec.lean:676-683`:
```lean
def Decaps (p : ParameterSet) (dk : 𝔹 (768 * k p + 96))
    (c : 𝔹 (32 * (dᵤ p * k p + dᵥ p))) : Option (𝔹 32) :=
  if Decaps.KeyCheck p dk then some (Decaps_internal p dk c)                      -- Alg. 21, steps 1–2
  else none
```
(comment lines omitted), with `Decaps.KeyCheck` (Spec.lean:666-669) `H ek = h  -- Eq. (7.2)` and `Decaps_internal` (Spec.lean:593-604) following FIPS 203 Alg. 18 steps 1-12 line by line, ending `if c ≠ c' then «K̄» else K'`. The `.MemoryAllocationFailure` arm concludes `out_of_memory`, which is `axiom out_of_memory : Prop` (`Symcrust/Properties/Axioms/System.lean:98`), documented as an "Opaque OOM witness"; the catch-all arm is `| _ => False`.

Strongest premise of decapsulate: `wfDecapKey` (`Symcrust/Properties/MLKEM/Sampling/ExpandMatrix.lean:1253-1257`):
```lean
structure wfDecapKey (self : mlkem.key.Key) (p : ParameterSet) : Prop extends wfEncapKey self p where
  has_private_key : self.has_private_key = true
```
which extends `wfEncapKey` (ExpandMatrix.lean:1209-1231):
```lean
structure wfEncapKey (self : mlkem.key.Key) (p : ParameterSet) : Prop extends Bridges.wfKey self p where
  hash_pinned : keyHashPinned self p
  byte_form_t : ∃ v_t : MLKEM.PolyVector q (k p),
    keyEncodedTPrefix self p =
      (MLKEM.PolyVector.ByteEncode 12 v_t).cast
        (Bridges.polyVector_byteEncode_size_cast 12)
    ∧ Bridges.keyT self p = v_t
  matrix_form_a : ∀ (i j : ℕ)
                    (hi : i < (k p : ℕ)) (hj : j < (k p : ℕ)),
    Bridges.keyAHat self p ⟨i, hi⟩ ⟨j, hj⟩ =
      MLKEM.SampleNTT
        (self.public_seed.toSpec ‖
          #v[(i : Byte)] ‖ #v[(j : Byte)])
```
(docstrings omitted), which extends `wfKey` (`Symcrust/Properties/MLKEM/Bridges/KeyView.lean:75-87`): `params_ok : wfInternalParams self.params p`, `n_rows_ok : self.n_rows.val = (k p : ℕ)`, `data_wf : ∀ i (h_end : i < dataEnd p), wfPoly (self.data.val[i]' ...)`. The theorem's comment says the dk validation required by FIPS 203 §7.3 is enforced "at key-creation- or key-load-time, which both establish the wfDecapKey condition".

`mlkem.encapsulate.spec` (Encaps.lean:656 ff.): premise `wfEncapKey`; `NoError` concludes `∃ (tape : MLKEM.RandomTape) ..., match MLKEM.Encaps params (pk_mlkem_key.toPubKey params) tape with | some (K, c, _) => K = ... ∧ c = ... | none => False`. `MLKEM.Encaps` (Spec.lean:650-660) reads 32 bytes from the tape and returns `none` iff `Encaps.KeyCheck` (ByteEncode₁₂∘ByteDecode₁₂ round-trip, Spec.lean:640-644) fails. `RandomTape` is `def RandomTape := ℕ → Byte` (Spec.lean:612).

`mlkem.key_generate.spec` (KeyGen.lean:261 ff.): premise `wfKey`; `NoError` concludes `wfPrivateSeed key' params ∧ ∃ (tape : MLKEM.RandomTape), let (ek, dk, _) := MLKEM.KeyGen params tape; ek = key'.toEncapKey params ∧ dk = key'.toDecapKey params`. `wfPrivateSeed` (`Symcrust/Properties/MLKEM/Encoding/KeySetValue/Prelude.lean:72`) extends `wfDecapKey` per the ExpandMatrix.lean:1179 docstring; its body was not quoted.

The tape existential is fed by the trusted axiom `random.spec` (`Axioms/System.lean:45-52`), which fixes `err = .NoError` and states `∃ (tape : Spec.MLKEM.RandomTape), ∀ (i : ℕ) (h_i : i < s'.length), s'.val[i] = tape i`.

`ShakeState.extract.spec` (`Symcrust/Properties/SHA3/StatefulXof.lean:71-84`): premise `absorbing self.state g ∨ squeezing self.state g` (ghost-state machine); conclusion `r.2.val = (extractOutput g result.length).toList` plus the next ghost state. The definitions of `absorbing`, `squeezing` and `extractOutput` were not quoted in this dossier.

Plain-language reading (facts about the statements, no verdict):
- Guaranteed, in the Lean model at d71d2e3f and given the trusted base in section 6: for any key satisfying the well-formedness structure, each public ML-KEM call terminates in the model without any Aeneas `Error` (no panic, overflow, out-of-bounds, assertion failure) and without `div`; on `NoError` the output bytes equal those computed by the Lean spec functions transcribing FIPS 203 Algs. 19-21; other error codes occur only in the listed arms (length mismatch, `out_of_memory`).
- For KeyGen and Encaps the statement is existential over the random tape: some tape exists that makes the output agree with the spec. Nothing is stated about the distribution or quality of randomness, and the RNG is axiomatised never to fail.
- Not guaranteed: fidelity of the Lean spec to the FIPS text (established by tests and review, not proof); correctness of Charon/Aeneas translation, rustc/LLVM and the C FFI layer; side-channel or constant-time behaviour; anything for keys that do not satisfy `wfDecapKey`/`wfEncapKey`/`wfKey` (those are established by other `key_set_value`/`key_generate` theorems, per comments); cryptographic security (IND-CCA).

## 4. Property classes covered
| Class | Status | Evidence |
|---|---|---|
| Functional correctness | yes | Top-level theorems equate Rust outputs with `MLKEM.Decaps/Encaps/KeyGen` and SHA-3 `extractOutput` (section 3.2). |
| Safety invariant | partial | Panic-freedom, absence of arithmetic overflow, in-bounds indexing, total termination (via `spec`); key well-formedness invariants (`wfKey`, `wfDecapKey`). Memory safety itself is inherited from safe Rust, not proved in Lean. |
| Liveness | partial | Termination only: `kyber_terminates` and total-correctness `spec`. No temporal liveness. |
| Refinement | yes | Implementation-to-spec refinement for each public function; an intermediate "auxiliary spec" layer is used [A1; C1]. |
| Symbolic security | no | Not addressed in any source read. |
| Computational security | no | "we verify only functional correctness, not leakage resistance" [A1 §3]; no IND-CCA proof. |
| Parsing | partial | Byte encode/decode and compress/encode functions are verified as part of ML-KEM; no wire-format protocol parsing. |
| Constant-time | no | "Side channels are out of scope at the Rust level: 'constant-time' code is verified for correctness but not leakage protection." [A1 §3] |
| Other | yes | Spec self-consistency: Keccak bijectivity, CAVP test vectors executed in Lean. Bugs found (section 9). |

## 5. Link to implementation code
Mechanism: extraction. Charon compiles the production Rust crate with rustc and exports MIR; Aeneas translates it to Lean definitions in `SymCRust/lean/Symcrust/Code/` (committed; regenerated by `make extract`). Proofs are written about these generated definitions. Guarantee: the theorems hold for the Aeneas-generated Lean model of the rustc MIR of the shipping code, for targets x86_64, i686 and aarch64 (`CHARON_OPTIONS=--targets=x86_64-unknown-linux-gnu,i686-unknown-linux-gnu,aarch64-unknown-linux-gnu`, EXTRACTION.md), conditional on the trust assumptions in section 6. The Rust code is not modified for verification except for a `verify` Cargo feature that redirects intrinsics to shims, opacifies FFI-calling code and excludes test code [A1 §7; C1]. INTRINSICS.md: "Without `verify`, `cargo build` is bit-identical to a tree with `src/verify/` removed." The C FFI layer is out of scope [A1 §7].

The paper also describes Eurydice (Rust to C via Charon/KaRaMeL) for legacy platforms; "the SymCrypt team plans to ship all compiled Rust code directly" [A1 §8.2].

## 6. Trusted computing base and soundness hygiene
Counts for `SymCRust/lean` at c2e575a (170 `.lean` files, 101,591 lines including the 30k-line generated `Code/`; `rg -w`):
| Token | Raw hits | Notes |
|---|---|---|
| sorry | 13 | 0 used as a tactic; all hits are in comments or docstrings (checked line by line). |
| admit | 1 | prose ("to admit an instance-specific definition"), not a tactic. |
| axiom | 117 hits; 90 `axiom` declarations | 34 in `Code/FunsExternal_Template.lean` and 1 in `TypesExternal_Template.lean` (generated templates); 21 in `Code/FunsExternal.lean` (opaque FFI/system functions: `try_new_box_zeroed`, `cpu_features_present`, `random`, `wipe_slice`, `InPlaceOrDisjointBuffer.*`, `const_time_*_impl`, `SymCryptMlKemSelftest`, Keccak4x `drop`); 13 spec axioms in `Properties/Axioms/` and `Properties/MLKEM/Axioms/BoxDefault.lean`; 20 intrinsics axioms in `Intrinsics/Axioms/` (AES rounds, PCLMULQDQ). |
| native_decide | 76 | Most in ML-KEM NTT/Montgomery and SHA-3 files; trusted per paper §3. |
| bv_decide | 60 | Trusted per paper §3. |
| implemented_by / extern | 0 / 0 | |
| opaque | 42 | Mostly `verify::opaque` comments and Lean `opaque`. |
| partial | 61 | Includes `partial_fixpoint` and prose. |

Selected trusted axioms, verbatim:
- `Symcrust/Properties/Axioms/System.lean:45` `axiom random.spec (s : Slice Std.U8) : common.random s ⦃ (err : common.Error) (s' : Slice Std.U8) => err = .NoError ∧ s'.length = s.length ∧ ∃ (tape : Spec.MLKEM.RandomTape), ...` (the RNG never fails, output modelled by an existential tape).
- `System.lean:98` `axiom out_of_memory : Prop`.
- `Symcrust/Properties/Axioms/Wipe.lean:28` `axiom common.wipe_slice.spec {T : Type} [Inhabited T] (s : Slice T) : common.wipe_slice s ⦃ (s' : Slice T) => s'.length = s.length ∧ ∀ i, (h : i < s'.length) → s'[i] = (default : T) ⦄`.
- `Symcrust/Properties/Axioms/Target.lean:28` `axiom theTarget : Std.Str`.
- Paper App. B: `axiom mm_aesenc_si128.spec r rkey : mm_aesenc_si128 r rkey ⦃ r' => r'.toBytes = Spec.AES.round r.toBytes rkey.toBytes ⦄`.

Aeneas Lean backend at main (275 files, 49,782 lines): 7 `axiom` declarations (`core.pin.Pin`, `PinHelper`, `PanicInfo`, `AtomicBool`, `AtomicU32`, `core.slice.Slice.get_unchecked_SliceIndexUsizeSlice_spec`, one in a test file); 35 `sorry` hits (not individually classified); 33 `native_decide`; 5 `implemented_by`; 1 `extern`.

Unverified translators: Charon (MIR to LLBC) and Aeneas (LLBC to Lean) are trusted. "We trust that Charon correctly extracts MIR into LLBC and that Aeneas correctly translates LLBC into Lean" [A1 §3]. The ICFP 2024 soundness results are pen-and-paper: "The proofs of the article by Ho et al. are only done with pen and paper; sometimes, the authors only sketch the reasoning over large case analyses" [A6 §1]. A Rocq mechanisation is ongoing ("approximately 10,000 lines of Rocq (excluding comments and blank lines)" [A6 §4]), covers "a very minimal subset of LLBC. It only contains integers and mutable references", and "found a flaw in the proofs of some of their fundamental lemmas. We repair their methodology using techniques from rewriting theory" [A6]. These results concern LLBC semantics, not the Aeneas implementation: "The actual symbolic interpreter relies on heuristics to select the rules" [A6 §2]. A6 §4 adds that the mechanised fragment "does not contain loops, it does not even contain if-then-else structures". Also trusted: rustc and LLVM back-ends, the Lean kernel, `native_decide`/`bv_decide`, Aeneas std-library models, intrinsic models, that callers meet preconditions, and that the Lean spec matches the standard [A1 §3; C1 §3].

Lean checker version and known kernel bugs. SymCrypt and the pinned Aeneas both use `leanprover/lean4:v4.31.0` [C1, C4]. The hax 0.4 release post states that "Lean v4.31.0, which has some known soundness bugs" (REF-L08 B5, [FULL]); it does not name the bugs. Two kernel soundness issues read in full:
- lean4#14484 (opened 2026-07-22, closed by #14498): "The checked kernel path for `Declaration.opaqueDecl` does not reject free or metavariables in the declaration body", allowing "an adversarial metaprogram" to prove `False` with `#print axioms` reporting no axioms [A7]. A maintainer comment (marked "AI generated") dates the omission to 2020-12-24, so v4.31.0 predates the fix; maintainers state that export-format/comparator checking cannot express the exploit ("those following the 'gold standard' for proof checking are not affected") [A7].
- lean4#14576 (opened 2026-07-28, fix #14577 listed under release v4.34.0): "Kernel accepts wrong-structure projections, allowing an axiom-free proof of False", reproduced on 4.34.0-nightly via `addDecl` in a metaprogram [A8]. The issue does not state the first affected version; whether v4.31.0 is affected is unknown.
Impact on this artefact: unknown. Both exploits need a metaprogram that calls `addDecl` with crafted terms; a lexical `rg addDecl` over `SymCRust/lean` at c2e575a returned no hits, but the Aeneas library, Mathlib and the `step`/`grind` tactics were not audited, and the development has not been replayed through an external checker (section 10). This is a lexical observation, not an audit. Separately, the 76 `native_decide` uses trust the Lean compiler and `Lean.ofReduceBool`, and the 60 `bv_decide` uses trust their LRAT-checking pipeline, independently of the kernel.

## 7. Scale, effort, automation
- Paper Fig. 4 totals: standard text 25,729 lines; spec core 2,827; spec properties 15,194; Rust 16,680; implementation properties 51,688; proofs 124,777. "the formalization comprises 237,160 lines of Lean, of which 30,253 are Aeneas-generated Lean definitions and 206,907 are source definitions and proofs." Per row: SHA-3 Rust 1,741 / spec core 140; ML-KEM Rust 2,349 / spec core 308 / properties 10,963 / proofs 25,389 [A1].
- Public README (cloc): SHA-3 Rust 1,656, spec 140, properties and proofs 15,211; ML-KEM Rust 2,125, spec 308, properties and proofs 37,989; intrinsics Rust 1,748, axioms 329, properties 5,177 [C1].
- Aeneas Lean backend: 29k lines per the paper (§5.2); 49,782 lines at main today. Aeneas OCaml sources: 74,868 lines (`src/*.ml`). Charon: 18 kLoC plus 9 kLoC hax-frontend-exporter, 2 person-years [A3 §2.5].
- Effort and AI: "it initially took us about six months to verify the NTT layer of ML-KEM whereas, at the time of writing, it takes about a week for an agent and a few days of human review to formalize, implement, and verify an additional algorithm." "agents generated ∼5.1k theorems for a total of 125 KLOC of proofs ... a few dollars per line of Rust code". Models: Claude Opus 4.6 (most proofs), then Opus 4.7, 4.8, GPT 5.6 Sol, Opus 5.0, via GitHub Copilot. On a 7-proof hard suite Opus 5 solved 7/7, mean 29.4 min and $15.4 per task [A1 §8.1].
- Automation: `step`/`step*` (monadic symbolic execution with `grind`), `scalar_tac`, `simp_scalar`, `simp_lists`, `bv_tac`, and the `#decompose` command [A1 §5.2-5.4].
- Build time: README says the development "is re-verified in about 15' by running `lake build` in `lean/`" [C1]. Not reproduced here.

## 8. Limitations stated by the authors (verbatim)
- "Aeneas has limited support for unsafe Rust and does not support raw pointers, interior mutability, or concurrency; functions using unsupported features must be modelled by hand." [A1 §5.1]
- Aeneas README (main): "`return` inside *nested* loops, or `break`/`continue` to *outer* loops are not supported yet"; "Instantiating a generic type (including `Self` through a trait implementation) with a mutable reference is not supported yet"; "The following limitations will be lifted by the ongoing work on separation logic: **unsafe code** **concurrency**" [C2].
- Charon: "some less-central features such as generic associated types, dyn Trait, or trait aliases are not yet fully supported" [A3 §2.6].
- "we verify only functional correctness, not leakage resistance." [A1 §3]
- "Models still try to 'cheat' with unsound assumptions or weakened goals, so careful human review of theorem statements remains essential." [A1 §8.1]
- "accepting that some of its code (e.g. its C FFI) is outside the scope of verification." [A1 §7]
- "Formal theorems apply only in a model of the Rust code and the system that uses it." [C1 §3]

## 9. Independent evidence
- Bugs found by verification [A1 §7]: (1) SHA-3 `extract` mid-lane bug producing wrong output bytes in multi-call SHAKE ("The SymCrypt team independently found this bug by differential testing"); (2) ML-KEM NTT runtime assertion `x<Q` should be `x≤Q` ("independently found by external audit"); (3) FrodoKEM constant-time comparison used `>> 8` instead of `>> 16` (experimental code).
- Adoption: shipping in Windows Insider builds [A4]; the SymCrypt team co-authored the blog [A4].
- Independent review: the Rocq mechanisation by Reynaud Michez (Verimag, JFLA 2026) of the LLBC soundness proofs, which found and repaired a flaw in the pen-and-paper proof method [A6]. No independent third-party audit of the SymCrypt Lean proofs was found.
- Performance (paper, same Skylake host): SHA-3 Rust 8% faster than SymCrypt C; ML-KEM Rust with AVX2 1.3-1.4× faster than SymCrypt C for encaps/decaps; Formosa/Jasmin 2-3× faster still [A1 §8.3].

## 10. Reproducibility
Not built: sandbox disk limit; queued for CI reproducibility job. At inspection time the sandbox had under 1 GB free disk and 2 CPUs; the build needs a Mathlib-backed Aeneas checkout and its cache. Exact commands (from README-VERIFIEDCRYPTO.md §4):
```bash
git clone -b feature/verifiedcrypto https://github.com/microsoft/SymCrypt
git clone https://github.com/AeneasVerif/aeneas && git -C aeneas checkout d71d2e3f2cb763f7faf15ab606ac9cf32da8dead   # adjacent to SymCrypt
cd SymCrypt/SymCRust/lean
lake exe cache get
lake build Symcrust
lake exe mlKemTests
lake env lean -e 'import Symcrust.Properties.MLKEM
open Symcrust.Properties.MLKEM
#print axioms mlkem.decapsulate.spec'
```
Static inspection only: clones at the commits in section 1 via `git clone --depth 1`.

## 11. Generalisation
Pattern: extrinsic verification of existing safe-Rust code by functional translation (Charon to Aeneas to Lean), proving refinement against an independently written, executable, standard-aligned Lean spec; AI agents write intermediate lemmas and proofs, humans review spec and top-level theorems.
To apply to an arbitrary codebase: the code must be in Rust and in Aeneas's supported subset (safe Rust; no interior mutability, raw pointers or concurrency; limited loop control flow and generic-with-`&mut` instantiation; Charon limits on GATs, `dyn Trait`, trait aliases). Unsupported functions must be marked `verify::opaque` and given trusted Lean axioms. Std-library functions need models. A Lean spec of the target standard must be written and validated. Platform intrinsics need models.
Case-specific preconditions: crypto primitives are sequential, deterministic (randomness abstracted as a tape), single-threaded, and have precise, published pseudocode. SymCrypt Rust is `no_std` and largely free of interior mutability. No network I/O, message interleaving, concurrency or adversary is modelled. Applying this to a protocol implementation (state machine plus I/O plus concurrency) would require the separation-logic extension the Aeneas README lists as ongoing, or hand-written models of I/O.

## 12. Bearing on sub-claims (facts only, no verdict)
| Sub-claim | Fact from this reference | Source ID |
|---|---|---|
| H1 | Lean specs of FIPS 202/203 are written as executable monadic code mirroring the standard; SHA-3 incremental API modelled with ghost state and a state machine (absorbing/squeezing); randomness modelled as an existential tape. No message-passing, concurrency or adversary model appears in the public release. | A1, C1 |
| H2 | ~5.1k theorems and 125 KLOC of proofs over 16.7 KLOC Rust; total-correctness theorems for public ML-KEM and SHA-3 functions; termination of rejection sampling proved. Properties are functional correctness, not protocol security. | A1, C1 |
| H3 | Mainstream-language link via extraction: the theorems concern the Aeneas model of rustc MIR of shipping Rust code; Charon and Aeneas are trusted, and the soundness proof of the underlying LLBC semantics is pen-and-paper and partially mechanised (with a flaw found and repaired). | A1, A3, A6, C1 |
| H4 | Route is Rust-only and restricted to Aeneas's safe-Rust subset; unsafe, interior mutability and concurrency unsupported. | A1, C2 |
| H5 | Implementation code was checked against Lean specs of algorithms without any protocol-level proof; HPKE composition (RFC 9180) verified functionally in experimental code, per the paper only. | A1 |
| D | "Verified" in SymCrypt = every public function has a theorem relating it to the spec, proofs complete, spec tested and reviewed. "Verify" covers safety (panic-freedom), functional correctness; explicitly not leakage resistance. | C1, A1 |

## 13. Conflicts
1. Termination status of `kyber_terminates`. `SampleNTTBridge.lean:108-110` says: "It is therefore fully proved — not assumed." `Symcrust/Properties/MLKEM/VERIFIED.md:425-432` (§6.2 "Almost-sure termination") describes it as almost-surely terminating with "no constructive bound is known, and the EasyCrypt ML-KEM proof leaves the same gap". A comment at `SampleNTTBridge.lean:304` calls it "(out-of-scope, axiomatic) `kyber_terminates`". The paper says it is proved via Barbosa and Schwabe [A1 §7.2]. [C1 internal]
2. ML-KEM size. Paper Fig. 4: Rust 2,349, properties plus proofs 36,352 [A1]. README: Rust 2,125, properties and proofs 37,989 [C1]. Different dates (paper September 2026, branch July 2026) and counting scripts.
3. NTT ring isomorphism and decryption-failure bounds. Paper §4: "Our formal development proves that the specified NTT realizes the intended ring isomorphism and establishes the claimed decryption-failure bounds" [A1]. A grep of the public branch's `Spec/` (`RingEquiv`, `≃+*`, failure terms) found no such theorem; the release may be a subset ("We have released a significant subset" [A1 Public Release]). Not resolved.
4. Aeneas supported subset. Aeneas README (main): nested loops are supported except `return` in nested loops and labelled `break`/`continue` [C2]. `docs/user/src/README.md` in the same repo: "loops: no nested loops for now", "no functions pointers/closures", "no nested borrows in function signatures" [C2]. The paper adds that it extended Aeneas with "a large class of nested (mutable) borrows" [A1 §1].
5. Paper Appendix C shows the `.MemoryAllocationFailure =>` branch empty; code has `out_of_memory` (Decaps.lean:1726-1730). Likely an HTML rendering loss.
6. `Result` and `spec` differ across Aeneas revisions. The paper describes an exception monad [A1 §5.1] and the pinned d71d2e3f defines `inductive Result ... | ok | fail | div` with `spec := theta` [C4]; Aeneas main c5eb4a2 defines `Result := ITree RustEffect` and a `DWP`-based `spec` [C2]. The Phase 1b version of this dossier quoted the main-branch definition as if it applied to SymCrypt; section 3.1 now separates the two.
7. Paper's description of soundness proofs vs. secondary account. A1 §3 lists Charon and Aeneas as trusted; A6 §1 states the ICFP 2024 proofs are pen-and-paper and that a "lacking argument ... could have invalidated their strategy" was found and repaired.

## 14. Sources
| ID | Title | URL | P/S | Depth | Summary |
|---|---|---|---|---|---|
| A1 | Scaling Verification of Cryptographic Software with Aeneas, Rust, and Lean (arXiv 2609.15648v1, draft TR, Sept 2026) | https://arxiv.org/html/2609.15648v1 | [P] | [FULL] | Methodology, Aeneas extensions, intrinsics modelling, SymCrypt case study, trust assumptions, AI-agent costs, bugs found, performance. |
| A3 | Charon: An Analysis Framework for Rust (CAV 2025, arXiv 2410.18042v2) | https://arxiv.org/html/2410.18042v2 | [P] | [FULL] | MIR-to-(U)LLBC driver, sizes, unsupported features, Aeneas and Eurydice as clients. |
| A4 | Verifying Rust cryptography in SymCrypt, from standards to code (MSR blog, 2026-07-13) | https://www.microsoft.com/en-us/research/blog/verifying-rust-cryptography-in-symcrypt-from-standards-to-code/ | [P] | [FULL] | Release announcement: ML-KEM and SHA-3 proofs; code used in Windows Insider builds; dashboards; agentic proofs. |
| A6 | Formal Verification of Borrow-Checking by Local Commutation Diagrams (Reynaud Michez, JFLA 2026) | https://hal.science/hal-05428143/document | [S] | [FULL] | Full PDF text (HAL v2, 31 Mar 2026) re-read in Phase 1c without an extractor. Rocq mechanisation in progress of the ICFP 2024 LLBC results; states those proofs are pen-and-paper; found and repaired a flaw; mechanised fragment has only integers and mutable references. |
| A7 | Missing check apparently allows proof of False after adversarial metaprogram (lean4 issue #14484, 2026-07-22) | https://github.com/leanprover/lean4/issues/14484 | [P] | [FULL] | Kernel `add_opaque` lacked the fvar/mvar closure check; fixed by #14498; maintainers note export-format checkers are unaffected. |
| A8 | Kernel accepts wrong-structure projections, allowing an axiom-free proof of False (lean4 issue #14576, 2026-07-28) | https://github.com/leanprover/lean4/issues/14576 | [P] | [FULL] | Reproduced on 4.34.0-nightly via `addDecl`; fix #14577 listed under release v4.34.0; first affected version not stated. |
| C1 | SymCrypt feature/verifiedcrypto @ c2e575a: README-VERIFIEDCRYPTO.md, SymCRust/lean (Spec, Symcrust/Properties, Intrinsics, EXTRACTION.md, VERIFIED.md, REPORT.md) | https://github.com/microsoft/SymCrypt/tree/feature/verifiedcrypto | [P] | [CODE] | Specs, top-level theorems, axioms, hygiene counts, build instructions, trust assumptions. |
| C2 | AeneasVerif/aeneas @ c5eb4a2 (README, docs/user, backends/lean) | https://github.com/AeneasVerif/aeneas | [P] | [CODE] | Supported subset (README, docs), main-branch `Result`/`spec` definitions, backend axioms and size. Not the revision SymCrypt pins. |
| C4 | AeneasVerif/aeneas @ d71d2e3f2cb763f7faf15ab606ac9cf32da8dead (2026-07-11), the revision pinned by SymCrypt EXTRACTION.md | https://github.com/AeneasVerif/aeneas/tree/d71d2e3f2cb763f7faf15ab606ac9cf32da8dead | [P] | [CODE] | `backends/lean/Aeneas/Std/Primitives.lean` (`Error`, `Result`) and `Std/WP.lean` (`theta`, `spec`, `dspec`, `⦃ ⦄` macros); `lean-toolchain` v4.31.0. Inspected via `git fetch --depth 1 --filter=blob:limit=1m`, then deleted. |
| C3 | AeneasVerif/charon @ f070bb9 | https://github.com/AeneasVerif/charon | [P] | [CODE] | Checked only for commit and layout. |

## 15. Unverified leads (excluded from evidence)
- A2 (moved in Phase 1c, abstract only): Aeneas: Rust Verification by Functional Translation (ICFP 2022), https://arxiv.org/abs/2206.07185. Abstract: "We leverage Rust's rich region-based type system to eliminate memory reasoning for many Rust programs, as long as they do not rely on interior mutability or unsafe code."
- A5 (moved in Phase 1c, abstract only): Sound Borrow-Checking for Rust via Symbolic Semantics (ICFP 2024), https://www.arxiv.org/abs/2404.02680. Abstract states that LLBC is proved sound w.r.t. a CompCert-style pointer model and that its symbolic semantics acts as a borrow-checker. The same content is cited in this dossier only through A6.
- Search results mention third-party commits bumping toolchains to v4.32.2 alongside the #14484/#14576 fixes, suggesting backports; no Lean release notes for v4.32.x were read, so the fixed-in versions below v4.34.0 are not established.
- Topic file 02 cites an Aeneas `documentation/crypto-verification.md` chain "Nist spec ⟷₁ Lean spec ⟷₂ Auxiliary spec ⟷₃ Aeneas translation"; the file exists in C2 but was not read in full here.
- `septract/aeneas-vcvio-demos` (linking Aeneas-extracted Rust to VCVio games) is mentioned in topic file 02; not inspected.
- The SymCrypt verification dashboards mentioned in A4 were not located.
- Paper reference to OpenAI "ten-proofs" (A1 ref [43]) not checked.
