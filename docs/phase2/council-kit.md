# Phase 2 Council Kit

Purpose: run a strong, reproducible multi-model debate on the owner's hypothesis, using only the Phase 1/1b evidence corpus. A capable orchestrator agent should be able to run the council from this document alone.

## Motion

> "LEAN4 cannot verify protocols AND therefore unable to check code implementation of these protocols." (owner, R1, verbatim; see [original-requests.md](../handoff/original-requests.md))

The council rules on each sub-claim first and then on the compound motion. A compound "true" requires both conjuncts and the inference (H5) to stand.

## Definitions

The council's first round (question D) must fix these before anything else. Each debater proposes definitions, and the judges adopt one set by majority, recording dissent. Candidate definitions, drawn from usage across the corpus:

| Term | Candidate readings to choose between |
|---|---|
| verify (a protocol) | (a) machine-checked proof of a stated property of a formal model; (b) also includes exhaustive or bounded model checking; (c) also includes testing |
| protocol | (a) cryptographic or security protocol (TLS, Noise, Signal); (b) also distributed or consensus protocols (Raft, Paxos); (c) also wire-format or parsing (MQTT, HTTP) |
| check code implementation | (a) proof that the code refines the spec; (b) proof about code translated into Lean (Aeneas/hax style), trusting the translator; (c) conformance testing of the code against a Lean executable spec (differential, trace validation); (d) runtime monitoring |
| any codebase | (a) any language and runtime; (b) any codebase in languages with an existing Lean bridge; (c) any codebase with a test harness or trace output |
| cannot | (a) impossible in principle; (b) not demonstrated to date; (c) impractical at realistic scale or cost |

The verdict must state which readings it used. Different readings legitimately produce different verdicts, and that is a valid outcome.

## Sub-claims

| ID | Sub-claim (from [CONVENTIONS.md](../../research/CONVENTIONS.md)) | Primary evidence sources |
|---|---|---|
| D | What "verify" and "check" mean | research/04, research/05, all dossiers §5 |
| H1 | Expressiveness | research/01, research/06, REF-L04..L06, REF-L18..L20 |
| H2 | Provability at scale | research/01, research/02, research/06, REF-L01..L12, comparators REF-C01..C06 |
| H3 | Implementation link | research/03, research/05, REF-L01..L03, REF-L07, REF-L08, REF-L13, REF-L14, REF-C01..C04, REF-C07..C09 |
| H4 | Generality across codebases | research/03, research/07, catalogue "Implementation-link patterns" |
| H5 | The "therefore" inference | research/05, REF-L01 (differential testing), REF-C07..C09 (conformance without proofs) |

## Roles and model families

Every role uses a different model family: six families in total ([ADR-0004](../adr/0004-council-model-diversity.md)). No judge shares a family with any advocate or with the red team. Families available to the orchestrator as of 2026-11-07 include Anthropic Claude, OpenAI GPT, Google Gemini, Moonshot Kimi, xAI Grok, DeepSeek and Zhipu GLM. Record the exact model IDs in the report.

| Role | Count | Brief | Family slot |
|---|---|---|---|
| Proposition | 1 (argues every sub-claim) | Argue each sub-claim is TRUE under the adopted definitions | A |
| Opposition | 1 (argues every sub-claim) | Argue each sub-claim is FALSE | B |
| Red team | 1 | Attack the strongest argument on each side; hunt for overclaims (AI-generated repos, sorry/axiom counts, unverified translators, lexical-only counts) | C |
| Judges | 3 | Score independently, then rule per sub-claim and write dissent | D, E, F |
| Clerk (orchestrator) | 1 | Enforce rules, keep the citation ledger, compile the report; does not vote | any |

Example assignment (illustrative, not binding): A = GPT, B = Gemini, C = Grok, D = Kimi, E = DeepSeek, F = GLM, clerk = Claude. The Phase 1 corpus was written mainly by Claude models, so the kit keeps Claude out of the advocate and judge seats to limit authoring bias. If fewer than six families are available, drop the red-team slot first and record the deviation.

## Rules of evidence

1. Only cite [FULL] or [CODE] entries from research/01-08 and research/references/, by ID (for example "REF-L04 §3", "research/06 A9"). Unverified leads are inadmissible.
2. Quote theorem statements and author scope statements verbatim. Do not paraphrase them into stronger claims.
3. Every "X proves Y" claim must state X's trusted base from the dossier's §6. Claims about AI-generated or unreviewed repos must carry that caveat.
4. Conflicts listed in [research/CONFLICTS.md](../../research/CONFLICTS.md) must be argued, not ignored. A side relying on one half of a conflict must address the other.
5. New evidence may be introduced only if it is fetched in full during the debate and appended to the ledger with URL and access date. The clerk may refuse it.
6. Absence of evidence is not proof of absence. Arguments of the form "not found" must say what was searched (see each file's gaps).

## Debate protocol

1. **Round 0, Definitions (D).** Each advocate proposes readings; judges adopt one set per term, with dissent recorded.
2. **Round 1, Openings.** For each of H1 to H5: proposition opens, then opposition (maximum 800 words each, citation-dense).
3. **Round 2, Rebuttals.** Each side rebuts the other's strongest point, citing evidence (maximum 500 words).
4. **Round 3, Red team.** The red team publishes overclaims found on both sides; each side gets one 300-word reply.
5. **Round 4, Judgement.** Each judge independently scores every sub-claim with the rubric, then rules TRUE / FALSE / PARTLY TRUE / UNDETERMINED, with a confidence of 0-100 and a one-paragraph reason.
6. **Round 5, Compound motion.** Judges rule on the full motion under the adopted definitions, plus at least one alternative definition set.
7. **Synthesis.** The clerk writes `phase2-council-report.md` using the template below. Majority rules; minority opinions are published in full.

Run advocates and judges in separate, isolated contexts. Judges see only the transcripts and the corpus, never the advocates' private reasoning.

## Scoring rubric

| Criterion | Weight | 0 | 3 | 5 |
|---|---|---|---|---|
| Evidence quality | 30% | uncited or leads only | some [FULL]/[CODE] citations | every material claim cited, verbatim quotes |
| Trust-base honesty | 20% | ignores TCB | mentions TCB | states sorry/axiom/translator assumptions for each claim |
| Engagement with conflicts | 15% | ignores | acknowledges | resolves or bounds each relevant CF entry |
| Definitional rigour | 15% | shifts definitions | consistent | shows how the result changes under other readings |
| Comparator grounding | 10% | Lean in isolation | some comparison | separates Lean-specific limits from field-wide limits (REF-C*) |
| Rebuttal strength | 10% | none | partial | answers the strongest opposing point directly |

## Evidence packets

Prepared inputs. Give every debater all of them, not a filtered subset.

| Packet | Contents |
|---|---|
| P0 Rules | this kit, [research/CONVENTIONS.md](../../research/CONVENTIONS.md) |
| P1 Reference catalogue | [research/references/README.md](../../research/references/README.md): property matrix, implementation-link patterns, TCB summary, sub-claim coverage |
| P2 Dossiers | research/references/REF-*.md |
| P3 Topic evidence | research/01-08 (evidence sections only) |
| P4 Conflicts | [research/CONFLICTS.md](../../research/CONFLICTS.md) |
| P5 Independent review | [research/REVIEW-phase1.md](../../research/REVIEW-phase1.md): known weaknesses of the corpus itself |
| P6 Benchmark corpus | research/07 protocol table (for Phase 3 pilot implications) |

## Output template

`phase2-council-report.md` must contain:

1. Council roster: role, model ID and family.
2. Adopted definitions (Round 0), with dissent.
3. Per sub-claim (D, H1-H5): ruling, mean confidence, the three judges' individual rulings, the key evidence cited by each side, and minority opinion.
4. Compound-motion ruling under the adopted definitions and under at least one alternative set.
5. Red-team findings, and which ones changed a ruling.
6. Evidence gaps that limited any ruling, as candidate Phase 3 research.
7. Implications for Phase 3: what may be built, under what stated assurance level, on which pilot protocol.
8. Citation ledger: every source ID cited, plus any new evidence admitted in Round 5 rule 5.
