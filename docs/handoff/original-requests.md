# Original Requests and Clarifications (verbatim)

This is the owner's instructions as given, in order. Agents must treat these as the authoritative statement of intent. Paraphrases elsewhere in the repo are subordinate to this file.

## R1 (2026-11-07): project start

> Create a new public GitHub repo using [https://github.com/bughunt8/skills/tree/main/skills/start-github-repo](https://github.com/bughunt8/skills/tree/main/skills/start-github-repo). Objective, enable protocol implementation verification on any codebase using LEAN4. The hypothesis is that LEAN4 cannot verify protocols AND therefore unable to check code implementation of these protocols. Perform initial research and store this into a ./research/ folder. plan to research many various aspects of this first and then Spawn multiple agents to gather all the relevant research. Do not analyse them yet, this will be a next stage which you need to write a phase1-research.md for handoff.md.

## R2 (2026-11-07): repository name

> name: Profit-Rise-Consulting/code-verification-lean4

## R3 (2026-11-07): Phase 1 review and correction

> three things: 1) copyright holder is Profit Rise Consulting;
> 2) are the objectives and actions and documents openly documented so that handoff has everything? review, fix and confirm; 3) **Your Goal** for Phase1 is to successfully enable Phase 2 Goal. Which is: perform a strong successful model council debate on the hypothesis. HENCE, Have a skeptical review of what work was completed, review the gaps in the research and thinking. Research can conflict, this will be part of the debate. EXCERPTS are not useful - the research should have actual information for Phase2 to be successful. There HAS ALREADY been work for existing protocols and algorithms -> THESE ARE REFERENCES that should be analysed, reviewed, generalized, documented: ALL READY for Phase2.

## Interpretation notes

The agent recorded these interpretations. The owner may overrule any of them.

| Ref | Interpretation | Where applied |
|---|---|---|
| R1 "Do not analyse them yet" | No verdict on the hypothesis in Phase 1. | research/CONVENTIONS.md rule 6 |
| R3 "references ... analysed, reviewed, generalized" | Prior works get a factual scope review: what is claimed, what is proved, the link to code, the TCB and how it generalises. This is distinct from judging the hypothesis, which is the council's job. | research/references/TEMPLATE.md |
| R3 "EXCERPTS are not useful" | A search snippet can locate a source but never counts as evidence. Anything not read in full is quarantined in "Unverified leads". | CONVENTIONS rule 1, scripts/check-research NFR-001 |
| R3 "Research can conflict" | Conflicts are preserved, not resolved, and aggregated in research/CONFLICTS.md as debate material. | CONVENTIONS rule 5 |
| R3 "strong successful model council debate" | Phase 2 is a structured multi-model debate with defined motion, definitions, roles, model-family diversity, evidence rules and rubric. | docs/phase2/council-kit.md |
