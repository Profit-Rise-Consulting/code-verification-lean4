# REF-L03: LNSym (Armv8 native-code symbolic simulator in Lean, with AWS-LC link)

## 1. Identity
| Field | Value |
|---|---|
| Subject (protocol or algorithm) | A Lean 4 model of a subset of the Armv8 AArch64 ISA, a symbolic-simulation proof framework for machine-code programs, and partial correctness proofs for cryptographic assembly routines taken from AWS-LC: SHA-512 block function (`sha512_block_armv8`), AES-GCM routines (`gcm_init_v8`, `gcm_gmult_v8`, `aes_hw_encrypt`, `aes_gcm_enc_kernel`) plus small examples (popcount, memcpy) [C1]. Cryptographic primitives at machine-code level, not a protocol. |
| Spec it targets (e.g. RFC 8446 section) | NIST FIPS 180-4 SHA-512 (Lean spec `Specs/SHA512.lean` "Closely follows the NIST standard"); AES/GCM specs `Specs/AESArm.lean`, `Specs/GCM.lean`, `Specs/GCMV8.lean`; Arm Architecture Reference Manual semantics for instructions [C1]. |
| Organisation / authors | Amazon Web Services; repo under `leanprover`. Lead: Shilpi Goel; contributors include Alex Keizer, Siddharth Bhat, Yan Peng, Nathan Wetzler, Leonardo de Moura [C1][S1]. |
| Primary URLs | https://github.com/leanprover/LNSym ; slides https://leodemoura.github.io/files/CAV2024.pdf |
| Repo, commit SHA, last commit date, licence | leanprover/LNSym @ `49f8a1d9e46f8dcfc345f4bc15d35b0ba845ff5c`, 2026-11-06 14:20:42 -0800, Apache-2.0 (bundled ELFSage parser under MIT, Draper Laboratory) [C1] |
| Lean toolchain (or other tool and version) | `leanprover/lean4:v4.35.0`; deps ELFSage (`shigoel/ELFSage` @ `lean-v4.35.0`) and doc-gen4; no Mathlib (a `FromMathlib.lean` excerpt instead). SAT automation via `bv_decide` (CaDiCaL + verified LRAT checker) [C1][S2]. |
| Maturity (research prototype / production / abandoned / AI-generated) | Research prototype, "under active development" in 2024 [S1]. Commit history shows a gap: substantive work to 2024-12-09, then a toolchain port from `nightly-2024-10-07` to `v4.35.0` landed 2026-11-06 (commits #254-#262) [C1]. Named authors. No claim of production use found. |
| AI provenance | `none-found`. Basis: full commit history (blob:none clone, 326 commits) has 130 `Co-authored-by` trailers, all naming humans (e.g. Goel, Siddharth, Keizer); the 2026 port commits are by Shilpi Goel and dependabot with no AI mention; no AGENTS.md, CLAUDE.md, .claude, .cursor or Copilot config at 49f8a1d [C1]. |

## 2. Claim as stated by the authors (verbatim)
- "LNSym is a symbolic simulator for Armv8 machine-code programs." [C1]
- "Focus: automated reasoning of cryptographic machine-code programs" [S1]
- "It uses Lean as ... A specification language to model the Arm instruction semantics and cryptographic protocols ... A theorem prover for reasoning" [S1]
- "Takeaways ... Lean programs are executable and efficient: conformance testing ... Automation: SAT, verified bit-blaster, and domain specific tactics implemented in Lean ... Interactive prover when automation fails" [S1]
- Lean release notes: `bv_decide` "is capable of handling complex, real-world problems, such as AWS's LNSym project for verifying cryptographic primitives implemented in ARM assembly." [S2]
- README: "Run `make` ... to fetch the Lean4 dependencies, build this library (including the proofs), and run conformance testing. Note that if you are not on an Aarch64 machine, conformance testing will be skipped." [C1]

## 3. What is actually formalised
Machine model: `ArmState` with GPRs, SIMD/FP registers, PC, error flag, flat byte memory, and the loaded `program` (map from address to 32-bit instruction word); `run n s` executes n fetch-decode-execute steps (`Arm/Exec.lean`, `Arm/State.lean`, `Arm/Decode`, `Arm/Insts/{BR,DPI,DPR,DPSFP,LDST}`) [C1]. No concurrency, interrupts, caches, exceptions beyond an error flag, or timing.

Programs are hand-entered instruction-word listings with provenance comments, e.g. `Tests/SHA2/SHA512Program.lean`:
```lean
-- sha512_block_armv8
-- Source: https://github.com/aws/aws-lc/blob/main/crypto/fipsmodule/sha/asm/sha512-armv8.pl#L454
def program : Program :=
  def_program
  [(0x1264c0#64 , 0xa9bf7bfd#32),      --  stp     x29, x30, [sp, #-16]!
```
Statement form: Hoare-style postconditions over `run k s0`, verbatim `Proofs/AES-GCM/GCMInitV8Sym.lean:32` (complete, no `sorry`) [C1]:
```lean
theorem gcm_init_v8_program_correct (s0 sf : ArmState)
    (h_s0_program : s0.program = gcm_init_v8_program)
    (h_s0_err : read_err s0 = .None)
    (h_s0_pc : read_pc s0 = gcm_init_v8_program.min)
    (h_s0_sp_aligned : CheckSPAlignment s0)
    (h_run : sf = run gcm_init_v8_program.length s0)
    (_h_mem : Memory.Region.pairwiseSeparate
      [ ⟨(H_addr s0), 128⟩,
        ⟨(Htable_addr s0), 2048⟩ ])
    : -- effects
      read_err sf = .None
    ∧ sf.program = gcm_init_v8_program
    ∧ CheckSPAlignment sf
    ∧ read_pc sf = read_gpr 64 30#5 s0
    ∧ Htable_addr sf = Htable_addr s0 + (9 * 16#64)
    ∧ H_addr sf = H_addr s0
    ∧ let Hinit := (read_mem_bytes 16 (H_addr s0) s0)
      read_sfp 128 20#5 sf =
      (GCMV8.GCMInitV8 ((BitVec.extractLsb' 0 64 Hinit) ++ (BitVec.extractLsb' 64 64 Hinit)))[0]!
```
The theorem covers one output register (v20); the memory conjuncts for the full Htable, register non-effects and memory-safety frame are commented out with "TODO: Commenting out memory related conjuncts since it seems to make symbolic execution stuck" (lines 59-75) [C1].

Proof status of the headline routines (theorem-level scan of `Proofs/` excluding `Experiments`, commit `49f8a1d`) [C1]:

| Theorem | File:line | Status |
|---|---|---|
| `gcm_init_v8_program_correct` | Proofs/AES-GCM/GCMInitV8Sym.lean:32 | complete (partial postcondition, see above) |
| `aes_hw_encrypt_program_run_60` | Proofs/AES-GCM/AESHWEncryptSym.lean:19 | complete, but the conclusion is only `read_err sf = .None` (60 steps run without a decode/fetch error); no claim that the output equals AES (section 3a) |
| `gcm_polyval_mul_eq_polynomial_mult`, `DPSFP.polynomial_mult_append` | Proofs/AES-GCM/GCMGmultV8Sym.lean:51,166 | `sorry` |
| `aes_gcm_enc_kernel_program_run_xx` | Proofs/AES-GCM/AESGCMEncKernelSym.lean:22 | `sorry` |
| `sha512_loop_sym` | Proofs/SHA512/SHA512BlockSym.lean:76 | `sorry` ("this theorem still ends in its pre-existing `sorry`", line 71) |
| `spec_simp_a` | Proofs/SHA512/SHA512BlockSym.lean:20 | `sorry` |
| `sha512_block_armv8_loop_2` | Proofs/SHA512/SHA512Loop.lean:186 | `sorry` |
| prelude theorem (16-step) | Proofs/SHA512/SHA512Prelude.lean:~120 | `sorry`, "PARKED (v4.19 port): this 16-step symbolic-simulation proof term triggers `(kernel) deep recursion detected` ... The full proof is preserved in git history (tag port-v4.18.0, where it still type-checked)" |
| `popcount32_sym_meets_spec` | Proofs/Popcount32.lean:76 | complete: `w0 sf = popcount32_spec (w0 s0)` plus register and memory frame conditions |

So no end-to-end "SHA-512 machine code equals FIPS spec for all inputs" theorem is complete at this commit; the SHA-512 work consists of per-basic-block lemmas (`Proofs/SHA512/SHA512LoopBlocks.lean`, 5,543 lines, 46 theorems) and loop/prelude theorems with `sorry` [C1].

### 3a. Predicate semantics: what the quoted theorems do and do not guarantee
Verbatim definitions, commit `49f8a1d` [C1]:

`Arm/Exec.lean:105-132` (abridged to the control structure)
```lean
def stepi (s : ArmState) : ArmState :=
  let err := read_err s
  match err with
  -- We take a step only if the starting state is error-free.
  | StateError.None =>
    (let pc := read_pc s
     let raw_inst := fetch_inst pc s
     match raw_inst with
      | none =>
        write_err (StateError.NotFound s!"No instruction found at PC {pc}!") s
      | some i =>
        let eai := decode_raw_inst i
        match eai with
          | none =>
            write_err (StateError.Unimplemented s!"Could not decode instruction {i} at PC {pc}!") s
          | some arm_inst => exec_inst arm_inst s)
  | _ => s

def run (n : Nat) (s : ArmState) : ArmState :=
  match n with
  | 0 => s
  | n' + 1 =>
    let s' := stepi s
    run n' s'
```
`Arm/State.lean:250` `def fetch_inst (addr : BitVec 64) (s : ArmState) : Option (BitVec 32) := s.program.find? addr`. `ArmState` docstring (`Arm/State.lean:156-158`): "Program: maps 64-bit addresses to 32-bit instructions. Note that we have the following assumption baked into our machine model: the program is always disjoint from the rest of the memory."

`Arm/Insts/Common.lean:211`
```lean
def CheckSPAlignment (s : ArmState) : Prop :=
  -- (FIXME) Incomplete specification: should also check PSTATE.EL
  -- after we model that.
  let sp := read_gpr 64 31#5 s
  (Aligned sp 4)
```
`Arm/Memory/Separate.lean:672` `def Memory.Region.pairwiseSeparate (mems : List Memory.Region) : Prop := mems.Pairwise Memory.Region.separate`.

What the theorems guarantee and what they do not:
- `read_err sf = .None` means: during the `n` steps, every PC hit an instruction in the `program` map that decoded to a modelled instruction, and no instruction set an error (`stepi` freezes the state once an error is set). It is a "no unmodelled or unfetchable instruction" property, not functional correctness. This is the whole conclusion of `aes_hw_encrypt_program_run_60` (`Proofs/AES-GCM/AESHWEncryptSym.lean:32`: `: read_err sf = .None := by`).
- All results describe the state after exactly `n` steps (`h_run : sf = run n s0`), with `n` chosen by the prover (27 for popcount, `gcm_init_v8_program.length` for GCM init, 60 for AES). They do not say the program terminates or returns for other inputs. `gcm_init_v8_program_correct` does add `read_pc sf = read_gpr 64 30#5 s0` (PC equals the link register, i.e. the function has returned).
- Instructions are fetched from `s.program`, a separate map, never from `mem`. Self-modifying code, code/data aliasing and instruction-fetch faults are outside the model by construction, and stores cannot change the code that is proved.
- `CheckSPAlignment` is a precondition, and its own comment marks it an "Incomplete specification" (exception level not modelled).
- `pairwiseSeparate` premises (e.g. H and Htable regions in GCM init; in, key and out regions in AES) are assumptions. The guarantees do not cover aliased buffers.
- `gcm_init_v8_program_correct` constrains only `v20` and two pointers; the full Htable memory contents and register non-effects are commented out (section 3). `popcount32_sym_meets_spec` is the only completed theorem here with full register and memory frame conditions (`REGS_UNCHANGED_EXCEPT`, `MEM_UNCHANGED_EXCEPT`, `Proofs/Popcount32.lean:88-89`).
- The model includes no timing, caches, interrupts or concurrency, so nothing is guaranteed about constant-time behaviour.


## 4. Property classes covered
| Class | Status | Evidence |
|---|---|---|
| Functional correctness | partial | Functional results proved: GCM init (one output register, v20) and popcount32. `aes_hw_encrypt_program_run_60` proves only error-free execution, not AES correctness. SHA-512 and GCM gmult/enc-kernel are incomplete (`sorry`) [C1]. |
| Safety invariant | partial | Postconditions include "no instruction decoding error", program unchanged, SP aligned [C1]. |
| Liveness | no | Bounded step counts `run n`; loops handled for one iteration (`h_N : N = 1#64`) [C1]. |
| Refinement | partial | Machine code vs Lean functional spec (`GCMV8.GCMInitV8`, `SHA2.*`) [C1]. |
| Symbolic security | no | |
| Computational security | no | |
| Parsing | tested | ELF loading via ELFSage for AWS-LC object files (test-only) [C1]. |
| Constant-time | no | No timing model. |
| Other | tested | ISA conformance testing of the Lean model against real Aarch64 hardware ("cosim") [C1][S1]. |

## 5. Link to implementation code
Two links, neither a verified compiler [C1]:
1. Program provenance (AWS-LC): the verified artefact is the machine code of AWS-LC's perlasm-generated assembly, entered as `(address, instruction-word)` lists with source comments pointing at `aws-lc/crypto/fipsmodule/sha/asm/sha512-armv8.pl` and similar. The `awslc_elf` make target builds AWS-LC in Docker (`scripts/ci_ubuntu_build_awslc.sh`) and `AWSLCELFTests` loads `sha512-armv8.S.o`, `aesv8-armx.S.o` etc. via ELFSage and `#guard_msgs`-checks the extracted instruction words (`Tests/ELFParser/AWSLCSHA2.lean:535-536`: `#eval do (getSymbolWords "sha512_block_data_order_hw" (← SHA512ELF))`). The comparison is against a recorded expected list, using `.o` files "to avoid relying on the linker-generated addresses" because PC-relative instructions differ after linking [C1]. Observed: the expected `.o` listing has `0x90000003#32` at the adrp position while the proof `program` has the linked word `0xd0000463#32` at `0x1264d4`, so the proved listing and the ELF-test listing are not syntactically identical at relocated instructions [C1].
2. ISA model validation: "cosim" conformance testing executes randomly generated instructions (`NUM_TESTS` "random tests/instruction class", default 20) on the host Aarch64 CPU and compares with the Lean model (`Arm/Cosim.lean`, `lake exe lnsym`); skipped on non-Aarch64 hosts [C1]. Slides: "Lean programs are executable and efficient: conformance testing" [S1].

Guarantee: proofs are about the Lean ISA model running a transcribed instruction list; faithfulness of the ISA model to silicon is supported by random testing, and correspondence to AWS-LC binaries by test-time ELF extraction. No claim was found that these proofs gate AWS-LC releases (AWS-LC's formal assurance is via SAW/s2n-bignum, outside this reference).

## 6. Trusted computing base and soundness hygiene
Counts over `.lean` files excluding `.lake`, non-comment lines [C1]:

| Token | Count | Location / note |
|---|---|---|
| sorry | 50 | Proofs (non-experimental) 12: SHA512 8 (Loop 5, BlockSym 2, Prelude 1), AES-GCM 4 (GmultV8 3, EncKernel 1); Proofs/Experiments 5; Tests 18; Benchmarks 10; Tactics 4; `Arm/Util.lean:9` 1 (a `FIXME` macro expanding to `sorry`, not used in Proofs). |
| admit | 0 | |
| axiom | 1 real (2 textual) | `Tactics/SkipProof.lean:39 axiom skipProofAx {α : Sort _} : α`, used by `skip_proof` (in `Proofs/Experiments/Memcpy/MemCpyVCG.lean`); second hit is its docstring. |
| native_decide | 194 | Mostly `Tests/` (AES/GCM/SHA/instruction tests); in model/spec/proof tree: `Arm/Insts/Common.lean` 4 (examples), `Specs/AESV8.lean` 3, `Specs/GCMV8.lean` 1, `Proofs/AES-GCM/GCMGmultV8Sym.lean` 2 (one commented), `Tactics/PruneUpdatesLogic.lean` 1. |
| implemented_by | 1 | `Correctness/Iterate.lean:119` (`iterate_impl`). |
| extern / opaque | 0 / 0 | |
| partial def | 53 | ISA decode/exec helpers (DPSFP 18), Tactics 16, Memory 7 and others. |

Additional TCB: `bv_decide` used 89 times in `Proofs/` (plus LRAT files in `Proofs/SHA512/lrat_files`); per the Lean 4.12.0 release blog, "Proofs generated by this tactic use the `Lean.ofReduceBool` axiom, so this tactic includes the Lean compiler as part of the trusted code base" [S2][C1]. ISA model correctness (tested, not proved); transcription of instruction words; ELFSage parser (tests only).

Porting regression: comparing with commit `5c05220` (2024-12-09, toolchain `nightly-2024-10-07`) via git history of a blob:none clone (`git show 5c05220:<file>`), `SHA512Prelude.lean` went from 0 to 1 `sorry` (the PARKED proof); SHA512Loop (5), SHA512BlockSym (2), GCMGmultV8Sym (3), AESGCMEncKernelSym (1) are unchanged [C1].

## 7. Scale, effort, automation
- Lean LOC at `49f8a1d`: 45,232 in 202 files; 665 `theorem`/`lemma` declarations [C1]. By directory: Arm (ISA model + memory reasoning) 14,334 (335 theorems); Specs 1,264; Proofs 12,522 (242); Tactics 6,149; Tests 9,266; Correctness 1,016; Benchmarks 464 [C1].
- Implementation verified: SHA-512 routine ~500 instructions ("Symbolically simulate (program.length - 16 - 3) = 485 instructions", SHA512Loop comment) [C1].
- Person-effort: not reported in sources read.
- Automation: custom symbolic-simulation tactics (`sym_n`, `sym_block`), `simp_mem`/`mem_omega` for memory separation, `prune_updates`, and `bv_decide` [C1][S1]. Performance limits are documented in code: the 485-step SHA-512 block proof needs `maxHeartbeats 0`; the 16-step prelude hits "(kernel) deep recursion detected" with "22 GB peak" at v4.35 [C1].
- Build time: not built (section 10).

## 8. Limitations stated by the authors (verbatim)
- "PARKED (v4.19 port): this 16-step symbolic-simulation proof term triggers `(kernel) deep recursion detected` under v4.19 — a kernel-performance limit ... NOT a logic error." (Proofs/SHA512/SHA512Prelude.lean) [C1]
- "(v4.35) still `(kernel) deep recursion detected` (22 GB peak) with main's body and maxRecDepth 100000; candidate for a rewrite (e.g. Lean's `vcgen`) instead." [C1]
- "TODO: Commenting out memory related conjuncts since it seems to make symbolic execution stuck" (GCMInitV8Sym.lean) [C1]
- "TODO: figure out all registers that are used ..." (non-effects frame not proved, GCMInitV8Sym.lean) [C1]
- "Note that if you are not on an Aarch64 machine, conformance testing will be skipped." [C1]
- Slides: "Interactive prover when automation fails" [S1].

## 9. Independent evidence
- Cited by Lean 4.12 release notes as a driver/use case for `bv_decide` [S2] and by Lean FRO talks [S1]; cited in later academic work as an example of low-level Lean verification (search results only; not read).
- Bugs found: current commits fix ISA model bugs found by new tests, e.g. "SMOV/UMOV: Rd = 31 is XZR ... Five of them fail on the old code" (commit `49f8a1d` message) [C1]. No reported bugs in AWS-LC from this project found.
- No external audit.

## 10. Reproducibility
Not built: sandbox disk limit; queued for CI reproducibility job. Exact command:
```
curl https://raw.githubusercontent.com/leanprover/elan/master/elan-init.sh -sSf | sh -s -- -y --default-toolchain none
git clone --depth 1 https://github.com/leanprover/LNSym && cd LNSym
timeout 1500 lake build Arm Specs Proofs   # or: make all VERBOSE=--verbose NUM_TESTS=20
make awslc_elf                             # needs Docker; builds AWS-LC then lake build AWSLCELFTests
```
Note the in-code report of a 22 GB peak for the parked prelude proof and `maxHeartbeats 0` on the SHA-512 block; cosim requires an Aarch64 host [C1]. Static facts were collected from a `--depth 1` clone plus `git show` reads of commit `5c05220` from a full-history blob:none clone.

## 11. Generalisation
Pattern: "verify the binary/assembly against a Lean ISA model": write an executable ISA semantics in Lean, validate it by co-simulation against hardware, transcribe the target machine code, and prove pre/postconditions by symbolic simulation with SAT-backed bit-vector automation.

To apply to an arbitrary codebase: needs an ISA model covering all instructions used (AArch64 subset here), a loader linking proofs to the shipped object code, memory-separation reasoning, and loop invariants. Case-specific preconditions: straight-line or simply-looping, constant-structure crypto kernels with fixed-size inputs; single-threaded; no OS interaction. Observed scaling limits: kernel performance on large symbolic-simulation terms (deep recursion, 22 GB) and toolchain drift (a proof that type-checked at v4.18 no longer does at v4.19+) [C1]. It does not cover protocol logic or higher-level C/Rust code.

## 12. Bearing on sub-claims (facts only, no verdict)
| Sub-claim | Fact from this reference | Source ID |
|---|---|---|
| H1 | Lean can express an executable AArch64 ISA model and pre/postconditions on machine code; no protocol state machine or concurrency. | C1, S1 |
| H2 | Headline SHA-512 and GCM-gmult/enc-kernel proofs contain `sorry` (12 in non-experimental Proofs). Complete with functional content: gcm_init_v8 (one register of output) and popcount32. aes_hw_encrypt is complete but proves only `read_err sf = .None`. All results are at a fixed step count, under a model where code is disjoint from data memory. | C1 |
| H3 | Proofs target AWS-LC assembly transcribed into Lean; ELF tests extract instruction words from AWS-LC `.o` files and compare with recorded lists; ISA model validated by random co-simulation on hardware. | C1 |
| H4 | Requires an ISA model per architecture; AArch64 host needed for conformance testing; tooling (SymSim tactics) specific to this model. | C1 |
| H5 | The machine code is checked against Lean functional specs (`Specs/SHA512.lean` etc.) independently of any protocol-level proof. | C1 |
| D | "Verifying cryptographic primitives implemented in ARM assembly" (Lean release notes) refers to a project whose main SHA-512 theorem is incomplete at this commit. | S2, C1 |

## 13. Conflicts
- Lean release notes describe `bv_decide` as handling LNSym's "verifying cryptographic primitives implemented in ARM assembly" [S2]; the repo at `49f8a1d` has the SHA-512 loop/prelude/block theorems ending in `sorry` [C1]. The block-level lemmas do use `bv_decide`, so both can be true (automation succeeded on blocks; end-to-end theorem incomplete). Recorded, not resolved.
- Instruction-word mismatch between proved `program` (linked addresses, e.g. `0xd0000463` adrp) and the ELF test listing from `.o` (`0x90000003`) [C1]; by the file's own comment this is an expected relocation difference, but it means the ELF test is not a mechanical check of the proved listing.

## 14. Sources
| ID | Title | URL | P/S | Depth | Summary |
|---|---|---|---|---|---|
| C1 | leanprover/LNSym repository @ 49f8a1d, plus git-history reads of commit 5c05220 (blob:none clone) (accessed 2026-11-07) | https://github.com/leanprover/LNSym | P | [CODE] | ISA model, symbolic simulation tactics, AWS-LC SHA-512/AES-GCM proofs (several `sorry`), cosim and AWS-LC ELF tests; counts in section 6. |
| S1 | Leonardo de Moura, "Lean 4: Bridging Formal Mathematics and Software Verification", CAV 2024 keynote slides (accessed 2026-11-07) | https://leodemoura.github.io/files/CAV2024.pdf | P | [FULL] | LNSym slide: Armv8 symbolic simulator led by Shilpi Goel; Lean as spec language and prover; conformance testing; SAT and verified bit-blaster. |
| S2 | Lean 4.12.0 release blog (archived) (accessed 2026-11-07) | https://web.archive.org/web/20250518080043/https://lean-lang.org/blog/2024-10-3-lean-4120/ | P | [FULL] (whole archived page re-read) | `bv_decide` introduced; cites LNSym; states use of `Lean.ofReduceBool` adds the Lean compiler to TCB. |

## 15. Unverified leads (excluded from evidence)
- Git tag `port-v4.18.0` is said (in code comments) to contain the complete SHA-512 prelude proof; no tag of that name (and no `port-*` tag) exists in a full-history blob:none clone, so the claim could not be checked.
- "Reference branch ... commit a0cdc07" mentioned in comments; not inspected.
- arXiv 2611.02028 cites LNSym (Goel et al. 2026); not read.
- Relationship to AWS-LC's own formal verification (SAW, s2n-bignum/HOL Light) not established from sources read; see AWS-LC dossier if any.
