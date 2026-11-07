# REF-L22: Perennial / Goose (Lean branch)

## 1. Identity
| Field | Value |
|---|---|
| Subject (protocol or algorithm) | Program verification of concurrent, crash-prone and distributed Go code. Go source is translated by the goose translator into GooseLang, a deeply embedded language, and verified in iris-lean separation logic. Lean-branch case studies: parts of the Go standard library (bytes, strings, sync, time, os, context, ...), `github.com/tchajed/marshal`, `github.com/goose-lang/primitive/std`, and goose testdata examples (channel, including an etcd-session example; crashfs; netfs; semantics; unittest; unsafememory) (S1, S4). |
| Spec it targets (e.g. RFC 8446 section) | The Go language semantics (as modelled) plus per-function separation-logic specifications written by the authors; no external standard. |
| Organisation / authors | MIT PDOS (mit-pdos). Recent `lean`-branch commits are by Frans Kaashoek and Upamanyu Sharma (S4, commit log of the shallow clone and GitHub history page header). |
| Primary URLs | https://github.com/mit-pdos/perennial (branch `lean`); Rocq development on branch `master` (S5). |
| Repo, commit SHA, last commit date, licence | `lean` @ 84d5380639a38f56cd517f1e81246fad7be5b21b, 2026-11-05 19:29:21 +0000. `git ls-remote --symref` reports `HEAD` → `refs/heads/lean`. No root LICENSE file was found on `lean`; `goose/LICENSE` exists (not read). Rocq `master` @ c3b7ecf (2026-10-25). Repository created 2019-03-02. |
| Lean toolchain (or other tool and version) | `leanprover/lean4:v4.34.1`; depends on iris-lean @ 6d6544a4394c1f89ce43b26a35f09de9b2158bc9 (subdir `Iris`). Translator: goose, written in Go. |
| Maturity (research prototype / production / abandoned / AI-generated) | Research. The Lean branch is under very active development, with many commits per day around the access date. The Rocq development is older and published (section 15). AI provenance: `agent-config-present`. `AGENTS.md` ("For every agent working in this repository, whatever the tool (Claude Code reads this file through `CLAUDE.md`; Codex and others read it directly)"), `.claude/` and `.agents/skills/{fix-semantics,land,review-trusted}` are present (S6). This shows that agents are expected to work in the repository. It does not show what share of the code they wrote; the shallow clone has no commit history to check for trailers. Evidence level: `static-inspected`. |

## 2. Claim as stated by the authors (verbatim)
- Lean README (S1, lines 3-6): "Perennial's goose framework and program proofs, in Lean 4 on top of iris-lean. This branch (`lean`) is the primary branch. It covers Go programs translated by goose, with a crash semantics and crash adequacy theorem but no crash program logic."
- goose README (S3, lines 3-12): "The translator is trusted: the process can be viewed as giving a semantics to Go. Goose supports a reasonable subset of Go, enough to write serious concurrent programs. GooseLang is an untyped lambda calculus with references and concurrency, with special support for Go's types, slices, maps, channels, interfaces and `defer`."
- Rocq master README (S5): "Perennial is a system for verifying correctness for systems with both concurrency and crash-safety requirements, including recovery procedures."
- `netfs_crash_safety` docstring (S4, `Perennial/Proof/github_com/mit_pdos/perennial/goose/testdata/examples/netfs/Results.lean:313-328`): "every boot program returns `#()` if it returns, no thread of any node is stuck and none has an unrecovered panic: `NodeA`'s check holds, however the network delays, duplicates, reorders or loses messages, whatever `"c"` sends, and however often any node crashes."

## 3. What is actually formalised
Pipeline (S1, S3, S4):
1. Go source → goose (Go program, 14,314 lines; 10,476 non-test lines excluding testdata) → Lean modules in `Perennial/Code/**`. Each function becomes a `noncomputable def F.impl : val` (a GooseLang term).
2. A per-package `class Assumptions : Prop` lists `F_unfold : FuncUnfold F [] F.impl` for each function, plus the imported packages' `Assumptions`. `FuncUnfold` is `func_unfold : #(functions f type_args) = f_impl` (`Perennial/Golang/Defn/PostLang.lean:127-128`). So the link "the name `F` denotes this body" is a hypothesis.
3. Go semantics: GooseLang (`Perennial/GooseLang`, 10,526 lines) together with Go-specific definitions (`Perennial/Golang/Defn`, 3,737 lines). The step relations for Go primitives are packaged as typeclasses: `go.Semantics` bundles `sem_fn`, `core_sem : go.PreSemantics`, `chan_sem`, `string_sem` and `pool_sem` (`Perennial/Golang/Defn.lean:21-26`). `PreSemantics` includes `array_sem` (`Golang/Defn/Pre.lean:34`).
4. Specs (`Perennial/Spec`, 18,494 lines in 521 files) are Iris Hoare triples. Proofs (`Perennial/Proof`, 62,393 lines in 748 files; `Perennial/GeneratedProof`, 59,677 lines emitted by `proofgen`) use iris-lean's WP and the Go-specific tactics `wp_start`, `wp_auto`, `wp_for`, ...
5. `Perennial/TrustedCode` (3,425 lines, 24 files) holds "trusted (hand-written) models of Go code" (README layering table, S1) for code goose does not translate, e.g. syscalls, `sync.Pool` Get/Put, and the toy network primitives.
6. Adequacy: `goose_crash_adequacy` (`GooseLang/CrashAdequacy.lean`), `goose_dist_adequacy` and `goose_dist_crash_adequacy` connect WP proofs to statements about executions of the semantics. The README says "There is no distributed program logic." (S1, lines 53-54)

### Worked trace: `netfs.recvFrom`
Go (`goose/testdata/examples/netfs/netfs.go:27-34`):
```go
func recvFrom(peer string) []byte {
	for {
		from, data := netext.Recv()
		if from == peer {
			return data
		}
	}
}
```
Translation (`Perennial/Code/github_com/mit_pdos/perennial/goose/testdata/examples/netfs.lean:37-41` and following; excerpt):
```lean
/-- recvFrom waits for a message from the node peer and returns its data,
    ignoring the messages of other nodes.

    go: netfs.go:27:6 -/
noncomputable def recvFrom.impl [GooseExts] [GoGlobalContext] : val :=
  (LamV "peer"
  (Let "$res0" (App (Val (GoInstruction (GoAlloc (go.GoType.SliceType go.byte)))) ...)
  (Seq (App (Val exceptionDo)
  (Let "peer" (App (Val (GoInstruction (GoAlloc go.string))) (Var "peer"))
  (App (App (App (Val doFor) (Lam BAnon
  (Val #true))) (Lam BAnon
  (Let "__p" (App (App (Val (GoInstruction (FuncResolve _root_.Perennial.github_com.mit_pdos.perennial.goose.testdata.examples.netfs.netext.Recv []))) (Val #())) (Val #()))
  ...
  (If (App (Val (GoInstruction (Convert go.untypedBool go.bool))) (App (Val (GoInstruction (GoOp GoEquals go.string))) (Pair (App (Val (GoInstruction (GoLoad go.string))) (Var "from")) (App (Val (GoInstruction (GoLoad go.string))) (Var "peer")))))
  ... (App (Val doReturn) ...
```
Every local variable is heap-allocated (`GoAlloc`) and read back with `GoLoad`. `for {}` becomes `doFor` with a constant-true condition, and the call goes through `FuncResolve netext.Recv`. The same file, lines 165-171, has `class Assumptions ... : Prop where recvFrom_unfold : FuncUnfold recvFrom [] recvFrom.impl ...`.

Spec (`Perennial/Spec/github_com/mit_pdos/perennial/goose/testdata/examples/netfs/recvFrom.lean:25-31`):
```lean
abbrev github_com.mit_pdos.perennial.goose.testdata.examples.netfs.recvFrom.spec : Prop :=
  ∀ (c0 peer : GoString), peer ≠ go!"c" →
    (netAddr (GF := GF) = go!"a" ∨ netAddr (GF := GF) = go!"b") →
    {{ isPkgInit (PROP := IProp GF) pkg_id.github_com.mit_pdos.perennial.goose.testdata.examples.netfs ∗
        inv netN (netInv (gooseNetGS (GF := GF)) c0) }}
      (App (Val (@! recvFrom)) (Val #peer))
    {{ (sl : GoSlice), RET #sl; sl ↦* c0 }}
```
Proof (`Perennial/Proof/.../netfs/recvFrom.lean:26-54`): `theorem ...recvFrom.wp_proof (wp_Recv : ...netext.Recv.spec) : ...recvFrom.spec`. It rewrites with `FuncUnfold.func_unfold`, then runs `wp_start`, `wp_auto`, `wp_for`, `wp_func_call`, `wp_apply_core wp_Recv'`, opens the protocol invariant (`imod netInv_recv`), and splits on `wp_if_destruct`. The comments on the two branches are "from `peer`: the data is `c0`" and "from another node: wait for the next message".

Link (`Perennial/Proof/.../netfs/Link.lean:29-30`, generated by `etc/gen-link.py`): `theorem ...netfs.wp_recvFrom : ...recvFrom.spec := ...recvFrom.wp_proof ...netext.wp_Recv`. `netext.Recv` is a trusted model, `λ: <>, ExternalOp netExt.name "Recv" #()` (`Perennial/TrustedCode/.../netfs/netext.lean:45`). Its behaviour is given by the network extension's step relation (`Perennial/GooseLang/Ext/Net/Impl.lean:71-72`).

Top-level result (`Results.lean:329-341`): `theorem netfs_crash_safety [lctx : GoLocalContext] [sem : go.Semantics] [package_sem : Assumptions] ...` (full statement in section 3a). It is proved from `netfs_crash_safety_bounded` (line 157), with step bound `n + 1` and thread bound `n + 2`. The netfs proof files total 1,084 lines.

## 3a. Predicate semantics
Conclusion of `netfs_crash_safety` (`Results.lean:329-341`, S4 @ 84d5380):
```lean
theorem netfs_crash_safety [lctx : GoLocalContext] [sem : go.Semantics]
    [package_sem : Assumptions]
    (σa σb σc : state) (g : GlobalState) (c0 old : GoString)
    (Ha : NodeStart lctx σa go!"a") (Hb : NodeStart lctx σb go!"b") (Hc : NodeStart lctx σc go!"c")
    (Hin : extLocal fsExt σa.world !! go!"in" = some c0)
    (Hout : extLocal fsExt σb.world !! go!"out" = some old)
    (Hsent : extLocal fsExt σb.world !! go!"sent" = some go!"")
    (Hneta : extGlobal netExt g.globalWorld !! go!"a" = some ∅)
    (Hnetb : extGlobal netExt g.globalWorld !! go!"b" = some ∅)
    (n : Nat) (κs : List Observation) (c : DistCfg)
    (Hsteps : DistCrashNsteps ((system σa σb σc).map Prod.fst) n
      (distInit (system σa σb σc) g) κs c) :
    ∀ (i : Nat) (tp : List Expr) (σ : state), c.nodes[i]? = some (tp, σ) →
      (∀ v tp', tp = Val v :: tp' → v = #()) ∧
      (∀ e, e ∈ tp → RealNotStuck e (σ, c.global)) ∧
      (∀ e, e ∈ tp → ¬ UnrecoveredPanic e)
```
Conclusion predicates (`Perennial/GooseLang/BoundedLang.lean:230-240`):
```lean
def RealNotStuck (e : Expr) (σ : CfgState) : Prop :=
  (toVal e).isSome ∨ ∃ κ e' σ' efs, RealPrimStep e σ κ e' σ' efs
inductive UnrecoveredPanic : Expr → Prop
  | panic (p : val) : UnrecoveredPanic (Val (PanicV p))
  | frame {Ki : EctxItem} {e : Expr} :
      Ki.unwinds = true → UnrecoveredPanic e → UnrecoveredPanic (fillItem Ki e)
```
Strongest premises:
- Execution relation (`Perennial/GooseLang/DistLang.lean:98-104`):
  ```lean
  inductive DistCrashNsteps (boots : List Expr) :
      Nat → DistCfg → List Observation → DistCfg → Prop
    | refl (c : DistCfg) : DistCrashNsteps boots 0 c [] c
    | cons {n : Nat} {c₁ c₂ c₃ : DistCfg} {κ κs : List Observation} :
        DistCrashStep boots c₁ κ c₂ → DistCrashNsteps boots n c₂ κs c₃ →
        DistCrashNsteps boots (n + 1) c₁ (κ ++ κs) c₃
  ```
- `NodeStart` (`Results.lean:62-66`): `goLctx : σ.goState.goLctx = lctx`, `packageState : σ.goState.packageState = ∅`, `threads : σ.threads = 1`, `addr : extLocal netExt σ.world = addr`.
- `[sem : go.Semantics]` (`Perennial/Golang/Defn.lean:21-26`, quoted in section 3) and `[package_sem : Assumptions]` (the `FuncUnfold` equations). `rg` found no `instance` of `go.Semantics`, `go.PreSemantics` or `ArraySemantics` in the repository. The theorem is therefore conditional on these classes being jointly satisfiable by some model; if they were contradictory, it would hold vacuously. That happened before: `docs/design/unsafe-memory-prerequisites.md` records "P1. `ArraySemantics` proves `False` (unsound)" via a disk-FFI axiom, so that "every result under it holds vacuously". It was resolved by removing the disk FFI (decision dated 2026-10-25). No current consistency proof (a model instance) was found.

Plain language. Guaranteed: take any finite-length execution of the three-node GooseLang system under the modelled crash and network semantics (any number of crashes, any message behaviour the network extension allows). In every reachable configuration, no thread of any node is stuck in the real semantics or has an unrecovered panic. So `NodeA`'s `panic("round trip mismatch")` is never reached. This holds assuming the goose output faithfully represents the Go functions, and the `go.Semantics` and `Assumptions` hypotheses. Not guaranteed: behaviour of the compiled Go binary; fidelity of the toy filesystem and network (README: "not a sound model of a Linux filesystem"; the network primitives "are stubs: the extension is a specification of a network"); the cases listed as unsound in `docs/KNOWN_LIMITATIONS.md` (section 8); progress or liveness; and functional correctness beyond the absence of a stuck thread or panic.

## 4. Property classes covered
| Property class | Covered? | Notes |
|---|---|---|
| Functional correctness | proved (conditional) | Per-function Iris Hoare triples for translated Go (stdlib, marshal, examples), under `go.Semantics`, `Assumptions` and trusted models (S1, S4). |
| Safety invariant | proved (conditional) | Crash safety (no stuck thread, no unrecovered panic) for crashfs and netfs via the crash and distributed-crash adequacy theorems (S1, S4). |
| Liveness | no | No liveness theorems found; the adequacy results are about finite executions. |
| Refinement | no | No refinement theorems on `lean`. Crash refinement reasoning is described for Rocq `master` only (S5). |
| Symbolic security | no | |
| Computational security | no | |
| Parsing / serialisation | partial | `tchajed/marshal` encode/decode functions have specs (14 spec files); no parser-soundness theorem. |
| Constant time / side channels | no | |
| Other | proved (conditional) | Concurrency: channels, `sync`, atomics, under the memory-model limits in section 8. |

## 5. Link to implementation code
The verified object is a translation of real Go source. The `.go` file is the code the developer compiles, and goose turns it into a GooseLang term. There is no extraction from Lean: the link runs from the Go code to Lean through a trusted translator (S3: "The translator is trusted: the process can be viewed as giving a semantics to Go"). Each generated `impl` keeps a Go source position comment (e.g. "go: netfs.go:27:6"). The schema's `impl_link` value `extraction` is used as the closest category, as REF-L07 (Aeneas) does for source-to-Lean translation. The direction (Go → Lean) and the trusted translator are stated in `tcb.unverified_translator`. Unmodelled library code is represented either by `axiom X : val` declarations (untranslated functions) or by hand-written `TrustedCode` models.

## 6. Trusted computing base and soundness hygiene
Trusted components (S1, S3, S4):
- The goose translator (Go; 10,476 non-test lines), and `proofgen` for `GeneratedProof`.
- The GooseLang and Go semantics definitions (`GooseLang`, `Golang/Defn`), including the crash relation. A crash "kills every thread, gives the restarted program an arbitrary heap and a fresh Go runtime (no package initialized), and crashes each extension's local state by the extension's own relation (`ExtSemantics.crash`); the global state is kept" (S1, lines 29-33).
- Typeclass hypotheses `go.Semantics`, `PreSemantics`, `ArraySemantics` and per-package `Assumptions`, with no instance found (section 3a).
- `TrustedCode` models (3,425 lines), five trusted package-variable initializers ("**Accepted for now, to remove later:** five such models are trusted although goose translates the real source", S1 lines 185-188), and `sync.Pool` Get/Put (S1).
- Extensions: a toy filesystem and a toy network, both specifications rather than models of an OS (S1).
- iris-lean and the Lean kernel.

Lexical counts (`rg -w` at 84d5380; lexical, not a proof-closure audit):
- `sorry`: 171 hits. 162 lines carry `sorry -- axiomatized type`, all in `Perennial/GeneratedProof`, e.g. `instance Replacer_typed_pointsto : TypedPointsto strings.Replacer := sorry -- axiomatized type` (`GeneratedProof/strings.lean:159-165`). One `admit` hit.
- `axiom` declarations: 3,726 lines matching `^\s*(noncomputable )?(private )?axiom `. Roughly 3,454 are `axiom X : val` and 103 are `: go.GoType` in `Perennial/Code`, standing for untranslated declarations; the largest files are `syscall` chunk0 with 1,907 and `runtime` chunk0 with 995. Some are `axiom T : Type`, for example for `sync.Map` and `os.Process`. Three are Prop axioms asserting specs: `time.Now.wp_proof` (`Proof/time/Now.lean:24`), `errors.As.wp_proof` (`Proof/errors/As.lean:25`) and `strings.Fields.wp_proof` (`Proof/strings/Fields.lean:26`). The last carries the docstring "FIXME: this is wrong (unsound) for strings with non-ASCII runes."
- `native_decide`: 0. `bv_decide`: 2 hits; README: "Do not use `bv_decide`/`native_decide`". `implemented_by`: 1; `extern`: 0; `opaque`: 27; `partial`: 88 (keyword hits, not all declarations).
- Whether `netfs_crash_safety` depends on any of these axioms or sorries was not determined; that needs `#print axioms`, which AGENTS.md prescribes ("To check whether a theorem is fully proved, run `#print axioms` on it and look for `sorryAx`"). No build was run.
- Audit tooling exists: `etc/lean-audit.py`, `etc/lean-audit.lean`, `etc/lean-ci.sh` and `etc/trusted-delta.py`, plus the `.agents/skills/review-trusted` skill. Their outputs were not inspected.
- Other model assumptions: a 64-bit platform (README lines 119-120), and executions of fewer than N steps and fewer than T live threads as explicit hypotheses of the bounded adequacy layer (README lines 98-104). `netfs_crash_safety` discharges these bounds for any `n`.

## 7. Scale, effort, automation
- Lean total: 310,390 lines in 1,640 files. Generated `Code` 115,096; `GeneratedProof` 59,677; `Proof` 62,393; `Spec` 18,494; `Golang/Theory` 24,220; `GooseLang` 10,526; `Std` 5,769; `Algebra` 2,597; `Ghost` 2,354; `TrustedCode` 3,425; `ProgramLogic` 1,085; `Golang/Defn` 3,737 (`wc -l`, S4).
- Automation: Iris proof mode plus Go-specific WP tactics. Overflow side conditions are discharged with `word`, `omega` and `bv_omega` (README line 116). Spec and proof files are separate per function, and `Link.lean` files are generated by `etc/gen-link.py`.
- Proof-checking times: not reported in the sources read, and not measured (no build).

## 8. Limitations stated by the authors (verbatim)
- "no crash program logic" (S1, line 6); "there is no crash WP (`wpc`)" (line 26); "There are no crash reasoning principles yet (no `crash_own`)" (lines 40-41).
- "There is no distributed program logic." (S1, lines 53-54)
- Toy filesystem: "not a sound model of a Linux filesystem" (S1, line 43). Network: "whose primitives are stubs: the extension is a specification of a network" (S1, line 62).
- `append` overflow assumption: "Known to be reachable, not yet fixed ... A program with that bug verifies." (S1, lines 148-151)
- KNOWN_LIMITATIONS (S2): "an entry marked **unsound** means a program with that behaviour can verify while Go does something else; check a program against these before relying on a proof of it." Entries marked unsound include:
  - map keys whose incomparable component sits behind a NaN field ("A key that `==` cannot test fully is accepted (unsound)");
  - entries added during `range`;
  - field or element addresses through `nil` that do not panic;
  - calling a nil func that does not panic;
  - a panicking deferred function that skips the rest of the chain;
  - `panic(nil)` treated as recovered;
  - a fixed evaluation order, e.g. `getF()(arg())` (accepted, not being fixed);
  - no allocation size limit;
  - unexported struct field names not package-qualified;
  - unsafe pointers outside an allocation not checked;
  - misaligned atomics assumed not to tear;
  - mixed-width atomics.
- Out of memory is outside the model (maxAddr 2^47), and the step bound is aggregated over nodes (S2).

## 9. Independent evidence
No independent replication or third-party audit of the Lean branch was found. The Rocq predecessor has peer-reviewed publications (SOSP 2019 Perennial, OSDI 2021 GoJournal, SOSP 2023 Grove, OSDI 2023 vMVCC), listed in the master README (S5). Those papers were not read for this dossier and are in section 15. They concern the Rocq development, not the Lean branch.

## 10. Reproducibility
Not built: sandbox disk limit; queued for CI reproducibility job. Exact commands:
```
git clone https://github.com/mit-pdos/perennial -b lean && cd perennial && git checkout 84d5380639a38f56cd517f1e81246fad7be5b21b
lake exe cache get   # if provided; otherwise full build
lake build Perennial.Proof.github_com.mit_pdos.perennial.goose.testdata.examples.netfs.Results
# then, in a Lean file importing it:
#print axioms netfs_crash_safety
```
The fully qualified name of `netfs_crash_safety` may carry the `Perennial.` namespace prefix. Facts in this dossier come from static inspection of a `--depth 1 --filter=blob:limit=1m` clone, which was deleted afterwards.

## 11. Generalisation
Scope axes: language = the subset of Go goose supports; runtime = the GooseLang model of the Go runtime (64-bit, bounded addresses); environment = toy filesystem and network extensions, plus syscalls via axioms or trusted models; build = the Go packages listed in goose configs. Outside the supported subset, or for code depending on unmodelled libraries, the method falls back to axioms or trusted models. Crash reasoning on `lean` is limited to adequacy, without a program logic, so per-function crash specs, as in the Rocq papers, are not available on `lean` (S1).

## 12. Bearing on sub-claims (facts only, no verdict)
| Sub-claim | Fact from this reference | Source ID |
|---|---|---|
| H1 | Go concurrency (goroutines, channels, sync), crashes with recovery and a multi-node network with message loss or duplication are expressed in a Lean deep embedding, with Iris specs for real Go code. Time is modelled through `time.Now`, which is an axiomatized spec. No probability. | S1, S4 |
| H2 | The netfs crash-safety theorem for a three-node system and specs for parts of the Go stdlib and marshal are stated in Lean (about 62k hand-written proof lines). They are conditional on semantic typeclasses with no instance found; 3 Prop axioms and 162 `sorry` instances for "axiomatized types" exist. Not built here. | S1, S4 |
| H3 | Go source (a mainstream language) is connected to Lean theorems through a trusted translator. The guarantee is stated relative to the model, and the authors list cases where "a program with that behaviour can verify while Go does something else". | S1, S2, S3 |
| H4 | The method is specific to Go (one translator per language). Untranslated stdlib and syscall code is represented by about 3,500 `axiom ... : val` declarations. | S3, S4 |
| H5 | The existence of a Go → Lean pipeline with conditional end-to-end crash-safety results bears on both implications (H5a, H5b): specifications and proofs are stated about translated code, not about a separate protocol model. | S1, S4 |
| D | "Verified" here = an Iris WP or adequacy theorem about the GooseLang translation under `go.Semantics` and `Assumptions`. "Crash safety" = no stuck thread and no unrecovered panic across crashes, not functional crash refinement. | S1, S4 |

## 13. Conflicts
- Default branch: `AGENTS.md` (S6) says "GitHub's default branch is `master`", while `git ls-remote --symref` on the access date reports `HEAD` → `refs/heads/lean`. The setting may have changed after AGENTS.md was written.
- Lean vs Rocq capability: the Rocq master README (S5) describes "crash weakest preconditions, crash invariants, idempotence, and crash refinement reasoning" in `src/program_logic/`. The Lean README (S1) says the Lean branch has "no crash program logic" and no `wpc`. Statements about Perennial's crash-reasoning capability therefore depend on which branch is meant. AGENTS.md calls the Rocq sources "frozen and out of date" and says "the Lean statements are authoritative".
- Soundness status of `ArraySemantics`: a design note dated 2026-10-24 records that it proved `False`. The 2026-10-25 decision removed the cause. At the inspected commit the defect is described as fixed, but no model instance was found to confirm consistency (section 3a).

## 14. Sources
| ID | Title | URL | P/S | Depth | Summary |
|---|---|---|---|---|---|
| S1 | Perennial README, branch `lean` @ 84d5380 (accessed 2026-11-07) | https://github.com/mit-pdos/perennial/blob/lean/README.md | P | [FULL] | Design, crash semantics, adequacy theorems, model assumptions, trusted code, conventions (321 lines). |
| S2 | docs/KNOWN_LIMITATIONS.md, branch `lean` @ 84d5380 (accessed 2026-11-07) | https://github.com/mit-pdos/perennial/blob/lean/docs/KNOWN_LIMITATIONS.md | P | [FULL] | List of known model-vs-Go disagreements, several marked unsound. |
| S3 | goose/README.md, branch `lean` @ 84d5380 (accessed 2026-11-07) | https://github.com/mit-pdos/perennial/blob/lean/goose/README.md | P | [SECT] opening section (lines 1-40). Quote: "The translator is trusted: the process can be viewed as giving a semantics to Go." | Translator role and the GooseLang description. |
| S4 | mit-pdos/perennial source, branch `lean` @ 84d5380639a38f56cd517f1e81246fad7be5b21b (accessed 2026-11-07) | https://github.com/mit-pdos/perennial/tree/lean | P | [CODE] @ 84d5380 | netfs Go source, translation, spec, proof, link, results; semantics typeclasses; counts. |
| S5 | Perennial README, branch `master` @ c3b7ecf (Rocq; accessed 2026-11-07) | https://github.com/mit-pdos/perennial/blob/master/README.md | P | [FULL] | Rocq development description, layout, publication list. |
| S6 | AGENTS.md, branch `lean` @ 84d5380 (accessed 2026-11-07) | https://github.com/mit-pdos/perennial/blob/lean/AGENTS.md | P | [SECT] "Notes for agents" bullet list and "Workflow: branch, review, land" steps 1-3. Quote: "Do not consult the Rocq sources (`master`, `new/`, `src/`, or any checkout of them such as `../etcd-grove-rr`). They are frozen and out of date, and the Lean statements are authoritative." | Agent instructions; branch policy; `#print axioms` advice. |

## 15. Unverified leads (excluded from evidence)
- Perennial (SOSP 2019): https://www.chajed.io/papers/perennial:sosp2019.pdf (not read).
- Goose (CoqPL 2020): https://www.chajed.io/papers/goose:coqpl2020.pdf (not read).
- GoJournal (OSDI 2021): https://www.chajed.io/papers/gojournal:osdi2021.pdf; proofs in mit-pdos/gojournal-proof (not read).
- Grove (SOSP 2023): https://pdos.csail.mit.edu/papers/grove:sosp23.pdf (not read).
- vMVCC (OSDI 2023): https://pdos.csail.mit.edu/papers/vmvcc:osdi23.pdf (not read).
- Tej Chajed PhD thesis: https://www.chajed.io/papers/tchajed-thesis.pdf (not read).
- Waddle MEng thesis: https://pdos.csail.mit.edu/papers/gibsons-meng.pdf (not read).
- GoTxn MEng thesis: https://pdos.csail.mit.edu/papers/mtheng-meng.pdf (not read).
- `docs/design/unsafe-memory-prerequisites.md` (lean branch): grepped for the P1 and P7 entries only, not read in full.
- Outputs of `etc/lean-audit.py` and `etc/trusted-delta.py`: not run.
- Branch `coq/tested` exists on GitHub; its contents were not inspected.
