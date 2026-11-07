# Architecture Vision

<!-- TOGAF ADM: Phase A. Combines the Architecture Vision with the Statement of Architecture Work. -->

## Business context
Profit Rise Consulting is building AI and assurance services for Hong Kong, the Greater Bay Area and Asia Pacific. A credible, evidence-backed answer to "can Lean 4 check that a protocol implementation matches its specification?" supports both a reusable verification capability and advisory work on secure systems.

## Stakeholders and concerns
| Stakeholder | Concern | Where it is addressed |
|---|---|---|
| Owner (Profit Rise Consulting) | Defensible verdict before investing in tooling | PRD §3, docs/phase2/council-kit.md |
| Council debaters and judges | Quotable evidence on both sides | research/, research/references/ |
| Future implementers | Executable handoff | handoff.md, phase1-research.md |

## Value proposition
The decision whether to build is based on evidence: prior work analysed against a common template, conflicts kept visible, and the hypothesis tested in parts.

## Vision statement
For any codebase that implements a protocol, a Lean 4 specification of that protocol can be used to establish a stated, honest level of assurance (proof, refinement, or conformance checking) about the implementation, with every trust assumption written down. Whether this vision is achievable is exactly what Phase 2 decides.

## Scope of architecture work
**In scope.** Evidence pipeline (Phase 1/1b), council design (Phase 2), pilot architecture candidates (Phase 3, after the verdict).

**Out of scope.** Commercial packaging; verification of cryptographic primitives from scratch.

## Constraints
Sandbox disk (no local Lean builds), credit-limited agent runs, public sources only.
