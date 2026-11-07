# Changelog

All notable changes to this project are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and this project
adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

<!-- Keep the Unreleased section at the top. Cutting a release is then a rename plus a new empty
     Unreleased section. Allowed headings, in this order: Added, Changed, Deprecated, Removed, Fixed,
     Security. Dates are ISO 8601. A withdrawn release is marked [YANKED]. -->

## [Unreleased]

### Added
- Phase 1c: [SECT] depth level, H5a/H5b split, scope axes, AI-provenance and result-evidence-level categories (research/CONVENTIONS.md).
- Reference dossiers REF-L21 (CCF Lean), REF-L22 (Perennial/Goose), REF-L23 (Telltale), REF-C10 (Aneris/Trillium/Grove/Fairneris), REF-C11 (verified-compilation boundary).
- research/CONFLICTS.md, research/CASE-COMPARISON.md, research/REVIEW-phase1.md with dispositions.
- Backlog spec docs/handoff/backlog.json, scripts/sync-issues, GitHub issues #2-#19.

### Changed
- All topic files and dossiers re-tagged to [FULL]/[SECT]/[CODE]; index JSON on SCHEMA v2 (proved/tested/assumed).
- Council kit: six distinct model families; per-ruling scope axes; H2 cost threshold.

## [0.1.0] - 2026-11-07

### Added
- Repository scaffold from bughunt8/skills start-github-repo (internal-tool profile, Apache-2.0).
- Phase 1 topic files research/01-08; Phase 1b full-read rewrites and reference dossiers REF-L01..L20, REF-C01..C09.
- Research gates: scripts/check-research, scripts/build-catalogue; ADR-0002 to ADR-0004.

### Removed
- Node/Cloudflare toolchain from the scaffold (ADR-0002).
