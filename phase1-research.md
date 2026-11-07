# Phase 1 Research: Handoff to Phase 2

Status: Phase 1 (evidence gathering) complete. No analysis has been performed and no verdict on the hypothesis is recorded anywhere in `research/`.

## Objective and hypothesis
- Objective: enable protocol implementation verification on any codebase using Lean 4.
- Hypothesis to test: Lean 4 cannot verify protocols, and therefore cannot check code implementations of those protocols.
- Phase 1 rule: gather sourced evidence only. Supporting and contradicting statements sit side by side. Source tags (supports / contradicts / neutral) describe what a source says, not our conclusion.

## Inventory
All access dates are 2026-11-07. Every claim carries a URL and a primary [P] or secondary [S] tag. Many sources were read only as search excerpts, and each file marks which.

| File | Scope | Size / sources (as self-reported) |
|---|---|---|
| research/01-lean4-protocol-modeling.md | Lean protocol models, state machines, temporal logic, consensus | 357 lines, 70-source register, 15 open questions |
| research/02-lean4-crypto-and-network-verification.md | Cedar, SymCrypt, zk, VCVio, tls13-lean, LeanMqtt, Veil, industry use | 284 lines, about 125 URLs, side-by-side section 8 |
| research/03-lean4-code-extraction-and-fidelity.md | Aeneas, hax, LNSym, FFI, Std.Do/mvcgen, Loom/Velvet, TCB | 407 lines, 58 indexed sources, 15 open questions |
| research/04-other-tools-protocol-verification.md | 17 comparator tools and protocol case studies | 463 lines, 108 register entries |
| research/05-implementation-conformance-methods.md | MBT, trace validation, fuzzing, automata learning, RV, oracles | 168 lines, about 120 URLs |
| research/06-lean4-language-limits-and-automation.md | IO/concurrency, trusted base, automation, releases to v4.36.0-rc2 | 191 lines, about 100 URLs |
| research/07-protocol-spec-formats-and-corpora.md | Spec formats; 17-protocol candidate corpus with models, implementations, CVEs | 145 lines, about 140 URLs |
| research/08-academic-literature-and-community.md | 61-entry annotated bibliography, researchers, repos | 464 lines, 120+ URLs |

## Known evidence gaps (agent-reported, to be closed or accepted in Phase 2)
- Source depth: many primary sources were read from search excerpts only. Anything Phase 2 relies on must be re-fetched in full. Files 03, 04, 06 and 08 flag this most.
- No Lean 3 protocol/TLA/consensus lineage found (01). No Veil code-extraction path or full Lean liveness proof of a message-passing consensus protocol confirmed (01).
- No Lean 4 models found for TLS, QUIC, SSH, Noise, WireGuard, Signal, Raft, TCP, BGP, DNS, Kerberos, OAuth, 5G, Bluetooth or EMV in file 07. File 02 lists tls13-lean, quiche, flare, libp2p-fv and VeriDNS. These two reports have not been reconciled; reconcile in Phase 2.
- No source found for "SampleCore" (probably SampCert) or for "IETF FMBF" (02, 07).
- No sound C-to-Lean or Go-to-Lean extractor found; no Move verification in Lean; no Aeneas/hax extraction-success rates or hax TCB statement (03). Lean compiler is not formally verified (03).
- Std.Do/IO/Task semantics for concurrency and nondeterminism not confirmed in primary docs; liveness, real-time evidence thin (06).
- Not found: eBay trace validation, Quint/Jepsen/Tamarin-to-code, Coq/QuickChick oracles, angr conformance paper (05).
- Not fetched: Bhargavan TLS 1.3 ProVerif paper, Signal and Noise Explorer papers, Dafny docs, P/Stateright limitation pages (04).
- Live Lean Zulip not searchable; only the public archive. GitHub star counts and push dates missing for about 9 repos (08).
- Several Lean repos are AI-generated with no independent review (02). Treat their claims as unverified.
- Recorded conflict, unresolved: web pages say miTLS is verified against RFC 8446, while the 2026 Everest retrospective says the TLS handshake implementation proof was not completed (04).
- Releases 4.25 to 4.27, 4.31 and 4.33 not read individually (06). Some quotes came through a page summariser, marked "(extractor)".

## Instructions for Phase 2 (analysis)
1. Re-verify load-bearing claims. For each, fetch the full primary source. Downgrade anything that cannot be verified.
2. Decompose the hypothesis into testable sub-claims: (a) Lean cannot express protocols (safety, liveness, concurrency, nondeterminism, adversary models); (b) Lean cannot prove protocol properties; (c) Lean cannot connect a model to implementation code; (d) (a)-(c) imply the conclusion. Map evidence from files 01-08 to each.
3. Build an evidence matrix: sub-claim x supporting / contradicting / absent, with source IDs and strength (primary and fully read beats excerpt).
4. Compare against the other tools (file 04) on the same axes, to separate "Lean-specific limit" from "field-wide limit".
5. Classify implementation-link routes (file 03, 05): proof-level extraction, translation validation, conformance testing against a Lean spec (Cedar-style), runtime monitoring. State the trust assumptions of each.
6. Select a pilot protocol from the file 07 corpus with a rationale. Use criteria such as existing formal model, small reference implementation, known CVEs.
7. Produce `phase2-analysis.md`: verdict on the hypothesis with confidence, a recommended architecture for "any codebase", and a risk list. Update PRD.md, TRD.md and docs/architecture from it.
8. Track every implementation gap in a GitHub issue until verified closed.

## Repo notes
- Scaffolded from bughunt8/skills start-github-repo (internal-tool profile, Apache-2.0). Validators pass.
- Node-oriented scaffold leftovers (vitest.config.ts, .nvmrc, npm CI steps) need replacing with Lean (`lake`) tooling.
- Remote hardening not yet applied (each needs separate approval): branch ruleset, secret scanning and push protection, Dependabot, CodeQL, private vulnerability reporting.
- PRD.md, TRD.md and docs/architecture still contain template TODOs.
