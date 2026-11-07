# Phase 1b reference-dossier prompt (2026-11-07)

Common body sent to agents d1-d8. Per-agent reference lists are in the table below.

> Repo: /home/user/workspace/code-verification-lean4 (public GitHub Profit-Rise-Consulting/code-verification-lean4). Objective: enable protocol implementation verification on any codebase using Lean 4. Hypothesis, to be debated by a model council in Phase 2: "LEAN4 cannot verify protocols AND therefore unable to check code implementation of these protocols." The owner said there HAS already been work on existing protocols and algorithms, and that these are references to be analysed, reviewed, generalised and documented so they are ready for Phase 2. You are building reference dossiers. Analysis here means a factual scope review of each reference: what it claims, what it actually proves, how it links to code, its TCB, scale and limitations, and how it generalises. It does NOT mean a verdict on the hypothesis.
> First read research/CONVENTIONS.md (binding rules, no excerpt-only evidence) and research/references/TEMPLATE.md. Skim the existing topic files research/01-08 for leads, but treat their entries as unverified unless you re-read the source.
> For each assigned reference write research/references/<ID>-<slug>.md following TEMPLATE.md exactly, all 15 sections. Read the actual papers in full (arXiv HTML/PDF, ePrint, ACM open access) and the actual source code. Clone repos with git clone --depth 1 into /tmp/<id>, never via the GitHub API. Quote real theorem statements with file:line. Where feasible, attempt a build: install elan via curl https://raw.githubusercontent.com/leanprover/elan/master/elan-init.sh -sSf | sh -s -- -y --default-toolchain none, then run `lake build` with a 25-minute cap. Skip the build for Mathlib-heavy repos if the cache cannot be fetched (`lake exe cache get`), and record the outcome either way.
> Also write research/references/_index/<ID>.json following research/references/_index/SCHEMA.json.
> Edit only your own files. No git commit or push. When done, message /root with one line per dossier: ID, file, FULL/CODE source count, and the single most important fact.

| Agent | Dossiers |
|---|---|
| d1_aws_lean | REF-L01 Cedar (cedar-spec Lean proofs, differential random testing vs Rust, SymCC/SymCert); REF-L02 SampCert; REF-L03 LNSym (and its aws-lc link) |
| d2_verselab_stack | REF-L04 Veil (framework plus its protocol case studies); REF-L05 Lentil (temporal/liveness); REF-L06 Loom and Velvet |
| d3_rust_extraction | REF-L07 Aeneas plus Microsoft SymCrypt Lean proofs (ML-KEM, SHA-3 etc.); REF-L08 hax Lean backend plus Cryspen libcrux |
| d4_lean_crypto | REF-L10 VCVio (plus Hopscotch); REF-L11 tls13-lean (plus pg-lean, same author); REF-L12 Clean / Verified-zkEVM circuit verification |
| d5_lean_network | REF-L13 LeanMqtt; REF-L14 Std.Http / LeanServer / stdhttp-verify; REF-L15 NyxFoundation libp2p-fv and formal-eth; REF-L16 GhostBuster, VeriDNS, quiche/flare Lean work |
| d6_lean_distributed | REF-L17 raft-lean-squad; REF-L18 leanda (Konnov) plus lean-tla / tla-plus-lean-workflow; REF-L19 leslie (Majumdar); REF-L20 verified-consensus (fradamt) plus iris-lean plus cslib Raft |
| d7_comparators_impl | REF-C01 Project Everest (miTLS, HACL*, EverCrypt, EverParse, 2026 retrospective); REF-C02 IronFleet plus Verus distributed systems; REF-C03 Verdi Raft; REF-C04 Fiat-Crypto plus AWS s2n-tls/aws-lc SAW |
| d8_comparators_symbolic | REF-C05 Tamarin (TLS 1.3, 5G AKA, EMV); REF-C06 ProVerif/CryptoVerif (Signal, TLS 1.3, WireGuard, Noise Explorer) |
| d9_comparators_conformance | REF-C07 Ivy QUIC specification-based testing; REF-C08 TLA+ trace validation (CCF, MongoDB, etcd); REF-C09 AWS lightweight formal methods (P language, ShardStore) |
