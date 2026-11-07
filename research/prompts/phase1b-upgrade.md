# Phase 1b upgrade prompt sent to r01-r08 (2026-11-07)

Phase 1b: full-read upgrade of your file. The owner rejected excerpt-based evidence: "EXCERPTS are not useful - the research should have actual information for Phase2 to be successful." The Phase 2 goal is a strong model-council debate on the hypothesis, and your file is evidence for it.
First read research/CONVENTIONS.md, which defines the binding rules and sub-claim IDs H1-H5 and D.
Then, for your own file only:
1. Every entry that is excerpt-only, snippet-only, "(extractor)" or otherwise not read in full: fetch the full primary source and rewrite the entry with substance. Substance means verbatim definitions and theorem statements, numbers, scope statements and limitation quotes. If you cannot read it in full, move it to a final "Unverified leads (excluded from evidence)" section.
2. For each Lean repository you rely on, inspect the code as CONVENTIONS rule 3 requires (clone into /tmp, then record SHA, toolchain, sorry/axiom/native_decide counts and LOC).
3. Tag every evidence entry with the sub-claim IDs (H1-H5, D) it bears on and with its depth [FULL] or [CODE].
4. Add a "Conflicts" section listing disagreements between sources, both sides, unresolved.
5. Close the gaps you reported in your Phase 1 summary where possible, and state which remain.
6. Keep the file neutral, with no verdict. Put a header line at the top recording the counts of [FULL] and [CODE] entries and of unverified leads.
Do not edit other files and do not run git. When done, message /root with: file path, FULL/CODE/lead counts, gaps that remain.
