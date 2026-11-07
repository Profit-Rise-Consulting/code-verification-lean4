# Phase 1c: fixes from the independent review (2026-11-07)

Sent to dossier agents d1-d9 and topic agents r01-r08 after research/REVIEW-phase1.md (GPT-family reviewer: 1 blocker, 9 major, 2 minor, plus a late major, F-013).

## Dossier agents (common)
1. Update every `_index` JSON to SCHEMA v2 (research/references/_index/SCHEMA.json):
   - property values must be proved | tested | assumed | partial | no | n/a;
   - add `impl_links` (all mechanisms, including mixed cases), `evidence_level` and `ai_provenance`.
   (F-012, F-013)
2. Add "3a. Predicate semantics": verbatim definitions of each headline theorem's conclusion predicate and strongest premises, with file:line, plus a plain-language statement of what is and is not guaranteed. (F-003)
3. Re-tag sources by depth: [FULL], [SECT] or [CODE] per CONVENTIONS. Everything else moves to §15. (F-001, F-002)
4. Use exactly one AI-provenance category per CONVENTIONS. (F-006)
5. Run scripts/build-catalogue and scripts/check-research, and fix errors that name your IDs.

## Topic agents (common)
1. Re-tag every entry [FULL], [SECT] or [CODE] per CONVENTIONS. [SECT] requires named sections and a verbatim quote. Move [PART], [EXT] and similar entries to Unverified leads. (F-002)
2. Use the AI-provenance categories. (F-006)
3. Do not move from a lexical count to a characterisation. (F-004)

## Specific assignments
| Agent | Extra |
|---|---|
| d3 | F-001: REF-L08 B4/B5. Re-read both Cryspen pages in full; correct authors and dates (Bhargavan, 2026-02-12; Blaudeau, 2026-10-01). F-007: anchor L07 `spec` semantics to the pinned Aeneas d71d2e3f. |
| d4 | F-003: expand REF-L10 `SecurityTarget` into its quantitative bound; state the role of ByteLaws/KeyDiscipline. |
| d5 | F-013: L16 impl_links. F-010: new dossier REF-L23 Telltale. |
| d2 | F-010: new dossier REF-L22 Perennial/Goose (Lean branch vs Rocq results). |
| d6 | F-010: new dossier REF-C10 Aneris/Trillium/Grove/Fairneris. |
| d7 | Missing-reference (lower priority): new dossier REF-C11, the verified-compilation boundary (CompCert/CakeML; what a compiler theorem transfers; the lean-compcert caveat). |
| d8 | F-013: C05 impl_links (Arquint bridge). F-011: build research/CASE-COMPARISON.md. |
| d9 | F-013: C07 safety is not PROVED; C08 liveness classification. |
| r01 | F-004: count tables in research/01. |
